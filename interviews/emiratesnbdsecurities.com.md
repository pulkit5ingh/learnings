# Emirates NBD Securities — Full Stack Interview Preparation

## Company Overview

- **Client:** ENBD (Emirates National Bank Dubai)
- **Website:** emiratesnbdsecurities.com
- **Location:** Dubai
- **Domain:** Banking, Securities, Financial Technology

## Role Focus

Full Stack Heavy:
- Backend: Node.js, Express.js, NestJS
- Frontend: React.js, Next.js
- Databases: MongoDB, PostgreSQL, TypeORM
- Infrastructure: Redis, Queues (BullMQ, Kafka), Docker, Kubernetes
- Security: JWT, OAuth, SSO, 3Scale API Gateway

---

# Section 1: Node.js

---

## Q1: What is the Node.js Event Loop and how does it work?

### Explanation

Node.js is single-threaded but handles concurrency through the event loop. The event loop continuously checks if there are tasks to execute.

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    NODE.JS EVENT LOOP                        │
│                                                              │
│   ┌─────────────┐                                            │
│   │   timers    │  ← setTimeout, setInterval callbacks      │
│   └──────┬──────┘                                            │
│          ▼                                                    │
│   ┌─────────────┐                                            │
│   │  pending    │  ← I/O callbacks deferred from prev loop  │
│   │  callbacks  │                                            │
│   └──────┬──────┘                                            │
│          ▼                                                    │
│   ┌─────────────┐                                            │
│   │    poll     │  ← Retrieve new I/O events (MAIN PHASE)   │
│   └──────┬──────┘                                            │
│          ▼                                                    │
│   ┌─────────────┐                                            │
│   │    check    │  ← setImmediate() callbacks               │
│   └──────┬──────┘                                            │
│          ▼                                                    │
│   ┌─────────────┐                                            │
│   │   close     │  ← socket.on('close') callbacks           │
│   │  callbacks  │                                            │
│   └─────────────┘                                            │
│                                                              │
│  Between each phase: process.nextTick() and Promises run     │
└──────────────────────────────────────────────────────────────┘
```

### Answer

The event loop has 6 phases. Each phase has a FIFO queue of callbacks to execute. When Node.js starts, it processes the entry script, then enters the loop:

1. **timers** — executes `setTimeout`/`setInterval` callbacks whose delay has passed
2. **pending callbacks** — executes I/O callbacks deferred to the next loop
3. **poll** — retrieves new I/O events; blocks here if queue is empty waiting for events
4. **check** — executes `setImmediate()` callbacks
5. **close callbacks** — fires close events

`process.nextTick()` and resolved Promises run between **every** phase transition, making them higher priority than `setImmediate()`.

### Code Example

```javascript
console.log('1: sync');

process.nextTick(() => console.log('2: nextTick'));

Promise.resolve().then(() => console.log('3: Promise'));

setTimeout(() => console.log('4: setTimeout'), 0);

setImmediate(() => console.log('5: setImmediate'));

console.log('6: sync');

// Output order:
// 1: sync
// 6: sync
// 2: nextTick        ← runs before Promises
// 3: Promise         ← microtask queue
// 4: setTimeout      ← timers phase
// 5: setImmediate    ← check phase
```

---

## Q2: What is the difference between process.nextTick(), setImmediate(), and setTimeout()?

### Answer

| | `process.nextTick()` | `Promise.then()` | `setImmediate()` | `setTimeout(0)` |
|---|---|---|---|---|
| Queue | nextTick queue | microtask queue | check phase | timers phase |
| Priority | Highest | 2nd | After poll | After poll |
| Use case | Defer within current op | Async continuations | After I/O | Delay at minimum |

### Code Example

```javascript
const fs = require('fs');

fs.readFile('file.txt', () => {
  // Inside an I/O callback:
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
  // Inside I/O: setImmediate ALWAYS runs before setTimeout
  // Output: immediate, timeout
});

// Outside I/O: order is non-deterministic
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```

---

## Q3: How does Node.js handle concurrency with a single thread?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│              NODE.JS CONCURRENCY MODEL                       │
│                                                              │
│  JavaScript Thread (single)                                  │
│  ┌────────────────────────────────────────────────────┐      │
│  │  Your code runs here — one thing at a time         │      │
│  │  No blocking I/O — uses callbacks/promises         │      │
│  └────────────────────────────────────────────────────┘      │
│                        │                                     │
│                        │ delegates I/O to                    │
│                        ▼                                     │
│  ┌────────────────────────────────────────────────────┐      │
│  │  libuv Thread Pool (default: 4 threads)            │      │
│  │  Handles: fs, crypto, DNS, compression             │      │
│  └────────────────────────────────────────────────────┘      │
│                        │                                     │
│                        │ OS handles                          │
│                        ▼                                     │
│  ┌────────────────────────────────────────────────────┐      │
│  │  OS Async I/O (epoll/kqueue)                       │      │
│  │  Handles: TCP/UDP sockets, pipes                   │      │
│  └────────────────────────────────────────────────────┘      │
└──────────────────────────────────────────────────────────────┘
```

### Answer

Node.js achieves concurrency through non-blocking I/O, not multi-threading. When your code calls `fs.readFile()`, Node.js hands the work to libuv (C++ layer), which either uses OS async I/O or a thread pool. The JavaScript thread is immediately freed to handle other requests. When the I/O completes, the callback is placed in the event loop queue and executed when the JS thread is free.

This means 1000 simultaneous HTTP requests don't need 1000 threads — they share one JS thread and each just waits for I/O asynchronously.

---

## Q4: What are Streams in Node.js?

### Answer

Streams process data piece by piece instead of loading everything into memory.

Four types:
- **Readable** — source of data (fs.createReadStream)
- **Writable** — destination (fs.createWriteStream)
- **Duplex** — both (TCP sockets)
- **Transform** — modify data as it passes (zlib.createGzip)

### Code Example

```javascript
const fs = require('fs');
const zlib = require('zlib');

// Stream a large file, compress it, write output
// Never loads the full file into memory
fs.createReadStream('large-file.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('large-file.txt.gz'))
  .on('finish', () => console.log('Done'));

// Without streams: readFile loads entire file into RAM
// With streams: processes in 64KB chunks
```

---

## Q5: What is clustering in Node.js?

### Answer

Clustering creates multiple Node.js processes (workers) that share the same server port. Each worker is a separate OS process with its own event loop and memory. The master process distributes incoming connections across workers using round-robin.

### Code Example

```javascript
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isPrimary) {
  const cpuCount = os.cpus().length;
  console.log(`Master ${process.pid} starting ${cpuCount} workers`);

  for (let i = 0; i < cpuCount; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died, forking replacement`);
    cluster.fork(); // auto-restart dead workers
  });
} else {
  http.createServer((req, res) => {
    res.end(`Handled by worker ${process.pid}`);
  }).listen(3000);
}
```

---

## Q6: What are Worker Threads in Node.js?

### Answer

Worker Threads run JavaScript in parallel threads within the same process. Unlike clustering (separate processes), workers share memory via `SharedArrayBuffer`.

Use clustering for I/O-bound work (HTTP servers).
Use Worker Threads for CPU-bound work (image processing, encryption, data parsing).

### Code Example

```javascript
// main.js
const { Worker } = require('worker_threads');

function runHeavyTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./heavy-task.js', { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}

// heavy-task.js
const { workerData, parentPort } = require('worker_threads');

// This runs in a separate thread — won't block the event loop
const result = heavyCPUOperation(workerData);
parentPort.postMessage(result);
```

---

## Q7: How do you handle errors in Node.js?

### Answer

Four layers of error handling:

```javascript
// 1. try/catch for synchronous and async/await code
async function fetchUser(id) {
  try {
    const user = await db.findById(id);
    if (!user) throw new Error('User not found');
    return user;
  } catch (err) {
    logger.error({ err, id }, 'fetchUser failed');
    throw err; // re-throw so caller can handle
  }
}

// 2. Promise .catch() for promise chains
fetchUser(id)
  .then(processUser)
  .catch(err => res.status(500).json({ error: err.message }));

// 3. EventEmitter error events
stream.on('error', (err) => console.error('Stream error:', err));

// 4. Process-level — last resort (should never be needed if above is correct)
process.on('uncaughtException', (err) => {
  logger.fatal(err, 'uncaughtException — shutting down');
  process.exit(1); // ALWAYS exit — process is in unknown state
});

process.on('unhandledRejection', (reason) => {
  logger.fatal(reason, 'unhandledRejection');
  process.exit(1);
});
```

---

## Q8: What is the difference between CommonJS and ES Modules?

### Answer

| | CommonJS (`require`) | ES Modules (`import`) |
|---|---|---|
| Syntax | `require()` / `module.exports` | `import` / `export` |
| Loading | Synchronous | Asynchronous |
| Resolution | Runtime | Static (compile-time) |
| Tree shaking | No | Yes |
| `__dirname` | Available | Not available (use `import.meta.url`) |
| Node.js | Default | Opt-in (`.mjs` or `"type":"module"`) |

```javascript
// CommonJS
const express = require('express');
module.exports = { myFunction };

// ES Module
import express from 'express';
export const myFunction = () => {};

// ES Module __dirname equivalent
import { fileURLToPath } from 'url';
import { dirname } from 'path';
const __dirname = dirname(fileURLToPath(import.meta.url));
```

---

## Q9: How does memory management work in Node.js?

### Answer

Node.js uses V8's garbage collector. Memory is divided into:

- **Stack** — primitive values, function call frames (automatically freed)
- **Heap** — objects, closures (managed by GC)
  - **New Space** — recently allocated objects, collected frequently (Minor GC)
  - **Old Space** — objects that survived New Space, collected less often (Major GC)

**Common memory leaks:**
```javascript
// 1. Global variables accumulate
global.cache = {}; // never GC'd

// 2. Event listeners not removed
emitter.on('data', handler); // if emitter lives forever, handler is never freed
emitter.removeListener('data', handler); // fix

// 3. Closures holding large data
function createHandler() {
  const largeData = Buffer.alloc(10 * 1024 * 1024); // 10MB
  return () => {
    // largeData is captured and never freed even if handler is kept
    console.log(largeData.length);
  };
}

// 4. Timers not cleared
const interval = setInterval(work, 1000);
// later:
clearInterval(interval); // must clear when done
```

---

## Q10: What is non-blocking I/O and why does it matter for Node.js?

### Answer

Non-blocking I/O means a thread initiates an I/O operation and immediately continues without waiting for the result. The result is delivered via a callback/promise when ready.

```javascript
// BLOCKING (bad in Node.js — freezes entire server)
const data = fs.readFileSync('large-file.txt'); // blocks for 500ms
// All other requests wait during this 500ms

// NON-BLOCKING (correct)
fs.readFile('large-file.txt', (err, data) => {
  // This runs when file is ready
  // Other requests are handled while file reads
});
// Execution continues immediately here

// Modern async/await (non-blocking under the hood)
const data = await fs.promises.readFile('large-file.txt');
```

In a Node.js HTTP server handling 1000 req/s: if each request does 100ms of I/O, blocking would mean 1000 × 100ms = 100 seconds serialized. Non-blocking means all 1000 complete in ~100ms total.

---

## Q11: What is the purpose of package.json scripts and how do you structure them?

### Code Example

```json
{
  "scripts": {
    "start": "node dist/main.js",
    "start:dev": "nodemon --watch src --exec ts-node src/main.ts",
    "build": "tsc -p tsconfig.build.json",
    "test": "jest --passWithNoTests",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "lint": "eslint src --ext .ts",
    "migrate": "typeorm migration:run",
    "migrate:revert": "typeorm migration:revert",
    "migrate:generate": "typeorm migration:generate"
  }
}
```

---

## Q12: How do you implement graceful shutdown in Node.js?

### Answer

Graceful shutdown lets in-flight requests complete before the process exits.

```javascript
const server = app.listen(3000);

async function gracefulShutdown(signal) {
  console.log(`${signal} received, shutting down gracefully`);

  // 1. Stop accepting new connections
  server.close(async () => {
    console.log('HTTP server closed');

    // 2. Close database connections
    await db.close();
    await redisClient.quit();

    // 3. Exit
    process.exit(0);
  });

  // 4. Force exit after 30s if graceful shutdown hangs
  setTimeout(() => {
    console.error('Forcing shutdown after timeout');
    process.exit(1);
  }, 30000);
}

process.on('SIGTERM', () => gracefulShutdown('SIGTERM')); // Docker stop
process.on('SIGINT', () => gracefulShutdown('SIGINT'));   // Ctrl+C
```

---

## Q13: What is middleware in the context of Node.js HTTP servers?

### Answer

Middleware is a function that sits in the request-response pipeline. It receives `req`, `res`, and `next`. Calling `next()` passes control to the next middleware.

```
Request → [Auth MW] → [Logger MW] → [Route Handler] → Response
              ↓ (if fails)
           401 Response
```

```javascript
// Middleware signature
function loggerMiddleware(req, res, next) {
  console.log(`${req.method} ${req.url} ${Date.now()}`);
  next(); // pass to next middleware
}

// Error middleware — 4 parameters
function errorMiddleware(err, req, res, next) {
  console.error(err);
  res.status(500).json({ error: err.message });
}
```

---

## Q14: How do you implement environment configuration in Node.js?

### Code Example

```javascript
// config.ts
import { z } from 'zod';
import 'dotenv/config';

const schema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string(),
  JWT_SECRET: z.string().min(32),
  JWT_EXPIRES_IN: z.string().default('7d'),
});

const parsed = schema.safeParse(process.env);

if (!parsed.success) {
  console.error('Invalid environment variables:', parsed.error.format());
  process.exit(1);
}

export const config = parsed.data;

// Usage
import { config } from './config';
app.listen(config.PORT);
```

---

## Q15: What is the difference between spawn, exec, execFile, and fork in child_process?

### Answer

| Method | Use Case | Streams | Shell |
|---|---|---|---|
| `spawn` | Long-running processes, large output | Yes | No |
| `exec` | Short commands, buffered output | No | Yes |
| `execFile` | Run executables directly | No | No |
| `fork` | Spawn Node.js child process | Yes | No |

```javascript
const { spawn, exec, fork } = require('child_process');

// spawn — streaming output
const ls = spawn('ls', ['-la', '/usr']);
ls.stdout.on('data', data => process.stdout.write(data));

// exec — buffered, good for small output
exec('git log --oneline -10', (err, stdout) => {
  console.log(stdout);
});

// fork — spawn another Node.js file, has built-in IPC
const child = fork('./worker.js');
child.send({ task: 'process', data: [...] });
child.on('message', result => console.log(result));
```

---

# Section 2: Express.js

---

## Q16: What is middleware in Express.js and how does the middleware chain work?

### Diagram

```
┌─────────────────────────────────────────────────────────────┐
│               EXPRESS MIDDLEWARE PIPELINE                   │
│                                                             │
│  Incoming Request                                           │
│       │                                                     │
│       ▼                                                     │
│  ┌──────────┐   next()   ┌──────────┐   next()             │
│  │  cors()  │──────────▶│ helmet() │──────────▶ ...        │
│  └──────────┘            └──────────┘                       │
│       │ (if res.send()                                      │
│       │  called here,                                       │
│       │  chain stops)                                       │
│       ▼                                                     │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Route Handler  app.get('/users', handler)           │   │
│  └──────────────────────────────────────────────────────┘   │
│       │                                                     │
│       ▼  (if error thrown)                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Error Middleware  (err, req, res, next)             │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Code Example

```javascript
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import { rateLimit } from 'express-rate-limit';

const app = express();

// Global middleware — applies to ALL routes
app.use(helmet());                    // security headers
app.use(cors({ origin: process.env.ALLOWED_ORIGINS?.split(',') }));
app.use(express.json({ limit: '1mb' }));  // body parser
app.use(express.urlencoded({ extended: true }));

// Rate limiting middleware
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                  // max 100 requests per window per IP
  standardHeaders: true,
  legacyHeaders: false,
});
app.use('/api/', limiter);

// Request logging middleware
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    console.log(`${req.method} ${req.url} ${res.statusCode} ${Date.now() - start}ms`);
  });
  next();
});

// Route-specific middleware
const requireAuth = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) return res.status(401).json({ error: 'Unauthorized' });
  try {
    req.user = jwt.verify(token, process.env.JWT_SECRET);
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
};

app.get('/api/profile', requireAuth, (req, res) => {
  res.json({ user: req.user });
});
```

---

## Q17: How do you structure a large Express.js application?

### Folder Structure

```
src/
├── app.ts               ← Express app setup (no listen here)
├── server.ts            ← server.listen + graceful shutdown
├── config/
│   └── index.ts         ← env vars validated with zod
├── routes/
│   ├── index.ts         ← mounts all routers
│   ├── auth.routes.ts
│   └── users.routes.ts
├── controllers/
│   ├── auth.controller.ts
│   └── users.controller.ts
├── services/
│   ├── auth.service.ts  ← business logic
│   └── users.service.ts
├── repositories/
│   └── users.repo.ts    ← DB queries
├── middleware/
│   ├── auth.middleware.ts
│   └── error.middleware.ts
├── models/              ← Mongoose schemas or TypeORM entities
└── utils/
```

### Code Example

```typescript
// routes/users.routes.ts
import { Router } from 'express';
import { UsersController } from '../controllers/users.controller';
import { requireAuth } from '../middleware/auth.middleware';
import { validateBody } from '../middleware/validate.middleware';
import { CreateUserDto } from '../dto/create-user.dto';

const router = Router();
const ctrl = new UsersController();

router.get('/',        requireAuth, ctrl.getAll);
router.post('/',       validateBody(CreateUserDto), ctrl.create);
router.get('/:id',     requireAuth, ctrl.getById);
router.patch('/:id',   requireAuth, ctrl.update);
router.delete('/:id',  requireAuth, ctrl.remove);

export default router;

// app.ts
import usersRouter from './routes/users.routes';
import { errorHandler } from './middleware/error.middleware';

app.use('/api/users', usersRouter);
app.use(errorHandler); // must be last
```

---

## Q18: How do you implement error handling in Express.js?

### Code Example

```typescript
// Custom error classes
class AppError extends Error {
  constructor(
    public message: string,
    public statusCode: number,
    public code?: string
  ) {
    super(message);
    this.name = 'AppError';
  }
}

class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 422, 'VALIDATION_ERROR');
  }
}

// Global error middleware (must have 4 params)
function errorHandler(err, req, res, next) {
  // Known operational error
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      error: { code: err.code, message: err.message }
    });
  }

  // Mongoose/TypeORM validation errors
  if (err.name === 'ValidationError') {
    return res.status(422).json({ error: { message: err.message } });
  }

  // JWT errors
  if (err.name === 'JsonWebTokenError') {
    return res.status(401).json({ error: { message: 'Invalid token' } });
  }

  // Unknown — don't leak internals
  console.error(err);
  res.status(500).json({ error: { message: 'Internal server error' } });
}

// Usage in controller
async function getUser(req, res, next) {
  try {
    const user = await userService.findById(req.params.id);
    if (!user) throw new NotFoundError('User');
    res.json(user);
  } catch (err) {
    next(err); // pass to error middleware
  }
}
```

---

## Q19: How do you implement input validation in Express.js?

### Code Example

```typescript
import { z } from 'zod';

// Define schema
const CreateUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  password: z.string().min(8).regex(/[A-Z]/).regex(/[0-9]/),
  role: z.enum(['admin', 'user', 'viewer']).default('user'),
});

// Validation middleware factory
function validateBody(schema: z.ZodSchema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({
        error: {
          code: 'VALIDATION_ERROR',
          details: result.error.flatten().fieldErrors,
        },
      });
    }
    req.body = result.data; // replace with parsed/coerced data
    next();
  };
}

// Usage
router.post('/users', validateBody(CreateUserSchema), usersController.create);
```

---

## Q20: How do you implement authentication with JWT in Express.js?

### Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                   JWT AUTH FLOW                             │
│                                                             │
│  1. LOGIN                                                   │
│  POST /auth/login { email, password }                       │
│       │                                                     │
│       ▼                                                     │
│  Verify credentials → sign JWT → return token               │
│                                                             │
│  2. PROTECTED REQUEST                                       │
│  GET /api/profile                                           │
│  Authorization: Bearer <token>                              │
│       │                                                     │
│       ▼                                                     │
│  Auth middleware: verify token → attach user → next()       │
│       │                                                     │
│       ▼                                                     │
│  Route handler: req.user is available                       │
└─────────────────────────────────────────────────────────────┘
```

### Code Example

```typescript
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';

// Login handler
async function login(req, res, next) {
  try {
    const { email, password } = req.body;
    const user = await userRepo.findByEmail(email);

    if (!user || !(await bcrypt.compare(password, user.passwordHash))) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    const accessToken = jwt.sign(
      { sub: user.id, email: user.email, role: user.role },
      process.env.JWT_SECRET!,
      { expiresIn: '15m' }
    );

    const refreshToken = jwt.sign(
      { sub: user.id },
      process.env.JWT_REFRESH_SECRET!,
      { expiresIn: '7d' }
    );

    // Store refresh token hash in DB
    await userRepo.saveRefreshToken(user.id, refreshToken);

    res.json({ accessToken, refreshToken });
  } catch (err) { next(err); }
}

// Auth middleware
function requireAuth(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  if (!token) return res.status(401).json({ error: 'No token provided' });

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JwtPayload;
    req.user = payload;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    res.status(401).json({ error: 'Invalid token' });
  }
}
```

---

# Section 3: NestJS

---

## Q21: What is NestJS and how is it different from Express.js?

### Answer

NestJS is an opinionated Node.js framework built on top of Express (or Fastify). It uses TypeScript, decorators, and Angular-inspired architecture.

| | Express.js | NestJS |
|---|---|---|
| Architecture | Unopinionated | Opinionated (MVC + DI) |
| TypeScript | Optional | First-class |
| Dependency Injection | Manual | Built-in IoC container |
| Structure | You decide | Modules/Controllers/Services |
| Decorators | No | Yes |
| Testing | Manual setup | Built-in testing utilities |
| Learning curve | Low | Medium |
| Best for | Simple APIs, flexibility | Enterprise, large teams |

---

## Q22: Explain the NestJS Module System.

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                  NESTJS MODULE SYSTEM                        │
│                                                              │
│  AppModule (root)                                            │
│  ├── imports: [UsersModule, AuthModule, DatabaseModule]      │
│  │                                                           │
│  │   UsersModule                                             │
│  │   ├── controllers: [UsersController]                      │
│  │   ├── providers:   [UsersService, UsersRepository]        │
│  │   ├── imports:     [TypeOrmModule.forFeature([User])]     │
│  │   └── exports:     [UsersService]  ← shared with others  │
│  │                                                           │
│  │   AuthModule                                              │
│  │   ├── imports: [UsersModule]  ← uses UsersService        │
│  │   └── providers: [AuthService, JwtStrategy]              │
└──────────────────────────────────────────────────────────────┘
```

### Code Example

```typescript
// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService], // make available to other modules
})
export class UsersModule {}

// app.module.ts
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRootAsync({
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        url: config.get('DATABASE_URL'),
        autoLoadEntities: true,
        synchronize: config.get('NODE_ENV') !== 'production',
      }),
      inject: [ConfigService],
    }),
    UsersModule,
    AuthModule,
  ],
})
export class AppModule {}
```

---

## Q23: What are Guards in NestJS?

### Answer

Guards determine whether a request is authorized to proceed. They implement `CanActivate` and return `true`/`false`.

```typescript
// jwt-auth.guard.ts
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from './decorators/public.decorator';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) { super(); }

  canActivate(context: ExecutionContext) {
    // Check if route is marked @Public()
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    if (isPublic) return true;
    return super.canActivate(context);
  }
}

// roles.guard.ts — RBAC
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// Usage
@Controller('admin')
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles(Role.ADMIN)
export class AdminController {
  @Get('stats')
  getStats() { ... }
}
```

---

## Q24: What are Pipes in NestJS?

### Answer

Pipes transform input data or validate it. They run before the route handler.

```typescript
// Built-in pipes
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number) {
  // id is guaranteed to be a number, not a string
  return this.usersService.findOne(id);
}

// Custom validation pipe using class-validator
import { validate } from 'class-validator';
import { plainToInstance } from 'class-transformer';

@Injectable()
export class ValidationPipe implements PipeTransform {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    if (!metatype || !this.toValidate(metatype)) return value;

    const object = plainToInstance(metatype, value);
    const errors = await validate(object);

    if (errors.length > 0) {
      const messages = errors.map(e => Object.values(e.constraints || {}).join(', '));
      throw new BadRequestException({ errors: messages });
    }
    return object;
  }

  private toValidate(metatype: Function): boolean {
    const types = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype as any);
  }
}

// DTO with decorators
import { IsEmail, IsString, MinLength, IsEnum } from 'class-validator';

export class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;

  @IsEnum(Role)
  role: Role;
}
```

---

## Q25: What are Interceptors in NestJS?

### Answer

Interceptors wrap request/response. Used for: logging, response transformation, caching, timeout handling.

```typescript
// Response transform interceptor — wraps all responses in { data: ... }
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map(data => ({ data, timestamp: new Date().toISOString() }))
    );
  }
}

// Logging interceptor
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const req = context.switchToHttp().getRequest();
    const start = Date.now();

    return next.handle().pipe(
      tap(() => {
        console.log(`${req.method} ${req.url} — ${Date.now() - start}ms`);
      }),
    );
  }
}

// Timeout interceptor
@Injectable()
export class TimeoutInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000), // throw TimeoutError if handler takes > 5s
      catchError(err => {
        if (err instanceof TimeoutError) {
          throw new RequestTimeoutException();
        }
        return throwError(() => err);
      }),
    );
  }
}

// Apply globally in main.ts
app.useGlobalInterceptors(new TransformInterceptor(), new LoggingInterceptor());
```

---

## Q26: How does Dependency Injection work in NestJS?

### Answer

NestJS has an IoC (Inversion of Control) container. You declare what a class needs via constructor parameters, and the container creates and injects the dependencies.

```typescript
// Service (provider)
@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly usersRepo: Repository<User>,
    private readonly configService: ConfigService,
    private readonly mailerService: MailerService,
  ) {}

  async create(dto: CreateUserDto): Promise<User> {
    const user = this.usersRepo.create({
      ...dto,
      passwordHash: await bcrypt.hash(dto.password, 10),
    });
    const saved = await this.usersRepo.save(user);
    await this.mailerService.sendWelcomeEmail(saved.email);
    return saved;
  }
}

// Controller uses the service
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  @HttpCode(201)
  create(@Body() dto: CreateUserDto) {
    return this.usersService.create(dto);
  }
}

// Custom provider — useful for external services
const providers = [
  {
    provide: 'REDIS_CLIENT',
    useFactory: (config: ConfigService) => new Redis(config.get('REDIS_URL')),
    inject: [ConfigService],
  },
];

// Inject custom provider
@Injectable()
export class CacheService {
  constructor(@Inject('REDIS_CLIENT') private redis: Redis) {}
}
```

---

## Q27: How do you implement JWT authentication in NestJS with Passport?

### Code Example

```typescript
// auth.module.ts
@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.registerAsync({
      useFactory: (config: ConfigService) => ({
        secret: config.get('JWT_SECRET'),
        signOptions: { expiresIn: '15m' },
      }),
      inject: [ConfigService],
    }),
  ],
  providers: [AuthService, JwtStrategy, LocalStrategy],
  exports: [AuthService],
})
export class AuthModule {}

// jwt.strategy.ts
@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(config: ConfigService) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      secretOrKey: config.get('JWT_SECRET'),
    });
  }

  async validate(payload: { sub: string; email: string; role: string }) {
    // This return value is attached to req.user
    return { id: payload.sub, email: payload.email, role: payload.role };
  }
}

// auth.service.ts
@Injectable()
export class AuthService {
  constructor(
    private usersService: UsersService,
    private jwtService: JwtService,
  ) {}

  async login(email: string, password: string) {
    const user = await this.usersService.findByEmail(email);
    if (!user || !(await bcrypt.compare(password, user.passwordHash))) {
      throw new UnauthorizedException('Invalid credentials');
    }
    const payload = { sub: user.id, email: user.email, role: user.role };
    return {
      accessToken: this.jwtService.sign(payload),
    };
  }
}

// Get current user decorator
export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);

// Usage in controller
@Get('profile')
@UseGuards(JwtAuthGuard)
getProfile(@CurrentUser() user: User) {
  return user;
}
```

---

## Q28: How do you handle exceptions in NestJS?

### Code Example

```typescript
// Built-in exceptions
throw new BadRequestException('Invalid input');        // 400
throw new UnauthorizedException('Login required');     // 401
throw new ForbiddenException('Access denied');         // 403
throw new NotFoundException('User not found');         // 404
throw new ConflictException('Email already exists');   // 409
throw new InternalServerErrorException();              // 500

// Custom exception filter
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    let status = HttpStatus.INTERNAL_SERVER_ERROR;
    let message = 'Internal server error';

    if (exception instanceof HttpException) {
      status = exception.getStatus();
      const res = exception.getResponse();
      message = typeof res === 'string' ? res : (res as any).message;
    }

    response.status(status).json({
      statusCode: status,
      message,
      path: request.url,
      timestamp: new Date().toISOString(),
    });
  }
}

// Register globally in main.ts
app.useGlobalFilters(new AllExceptionsFilter());
```

---

## Q29: What is the lifecycle of a NestJS request?

### Diagram

```
Incoming Request
      │
      ▼
 Middleware           (app.use — logging, CORS, body parsing)
      │
      ▼
 Guards               (canActivate — auth check, roles)
      │
      ▼
 Interceptors (pre)   (before handler — logging start, cache check)
      │
      ▼
 Pipes                (transform/validate request data)
      │
      ▼
 Route Handler        (controller method)
      │
      ▼
 Interceptors (post)  (after handler — transform response)
      │
      ▼
 Exception Filters    (if error thrown anywhere above)
      │
      ▼
 Response sent
```

---

## Q30: How do you implement caching in NestJS with Redis?

### Code Example

```typescript
// cache.module.ts
@Module({
  imports: [
    CacheModule.registerAsync({
      useFactory: (config: ConfigService) => ({
        store: redisStore,
        host: config.get('REDIS_HOST'),
        port: config.get('REDIS_PORT'),
        ttl: 60, // default TTL seconds
      }),
      inject: [ConfigService],
    }),
  ],
  exports: [CacheModule],
})
export class AppCacheModule {}

// Usage with decorator
@Controller('products')
export class ProductsController {
  @Get()
  @UseInterceptors(CacheInterceptor) // auto-caches based on route + query
  findAll() {
    return this.productsService.findAll();
  }
}

// Manual cache control
@Injectable()
export class ProductsService {
  constructor(@Inject(CACHE_MANAGER) private cache: Cache) {}

  async findById(id: string) {
    const cacheKey = `product:${id}`;
    const cached = await this.cache.get(cacheKey);
    if (cached) return cached;

    const product = await this.repo.findOneBy({ id });
    if (!product) throw new NotFoundException();

    await this.cache.set(cacheKey, product, 300); // cache 5min
    return product;
  }

  async update(id: string, dto: UpdateProductDto) {
    const updated = await this.repo.save({ id, ...dto });
    await this.cache.del(`product:${id}`); // invalidate cache
    return updated;
  }
}
```

---

# Section 4: React.js

---

## Q31: What are React Hooks and why were they introduced?

### Answer

Hooks (introduced in React 16.8) let functional components use state and lifecycle features that previously required class components.

**Why introduced:**
- Class components had confusing `this` binding
- Logic was hard to reuse between components (HOCs and render props were complex)
- Unrelated logic was grouped by lifecycle method (`componentDidMount` had both data fetching and event listener setup)

**Core hooks:**

| Hook | Purpose |
|---|---|
| `useState` | Local component state |
| `useEffect` | Side effects (data fetch, subscriptions) |
| `useContext` | Consume context |
| `useRef` | Mutable ref, DOM access |
| `useMemo` | Memoize expensive computation |
| `useCallback` | Memoize function reference |
| `useReducer` | Complex state logic |

```typescript
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let cancelled = false; // prevent state update on unmounted component

    async function fetchUser() {
      try {
        setLoading(true);
        const data = await usersApi.getById(userId);
        if (!cancelled) setUser(data);
      } catch (err) {
        if (!cancelled) setError(err.message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    fetchUser();
    return () => { cancelled = true; }; // cleanup
  }, [userId]); // re-run when userId changes

  if (loading) return <Spinner />;
  if (error) return <Error message={error} />;
  return <div>{user?.name}</div>;
}
```

---

## Q32: What is the difference between useMemo and useCallback?

### Answer

Both memoize values to prevent unnecessary recalculations/re-renders. The difference is what they return.

```typescript
// useMemo — memoizes a COMPUTED VALUE
const filteredUsers = useMemo(
  () => users.filter(u => u.role === selectedRole),
  [users, selectedRole] // recomputes only when these change
);
// Without useMemo: filter runs on every render even if users/selectedRole unchanged

// useCallback — memoizes a FUNCTION REFERENCE
const handleDelete = useCallback(
  async (id: string) => {
    await usersApi.delete(id);
    setUsers(prev => prev.filter(u => u.id !== id));
  },
  [setUsers] // function reference stable unless setUsers changes
);
// Without useCallback: new function created every render → child re-renders unnecessarily

// When useCallback matters: passing callbacks to React.memo'd children
const MemoizedButton = React.memo(({ onClick }: { onClick: () => void }) => (
  <button onClick={onClick}>Delete</button>
));

// This works correctly — handleDelete reference is stable
<MemoizedButton onClick={handleDelete} />
```

---

## Q33: How does Redux work and when should you use it?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    REDUX DATA FLOW                           │
│                                                              │
│  Component                                                   │
│    │  dispatch(action)                                       │
│    ▼                                                         │
│  Store.dispatch()                                            │
│    │                                                         │
│    ▼                                                         │
│  Middleware (redux-thunk / redux-saga)                       │
│    │  (async operations happen here)                         │
│    ▼                                                         │
│  Root Reducer  (pure function: (state, action) => newState)  │
│    │                                                         │
│    ▼                                                         │
│  New State                                                   │
│    │                                                         │
│    ▼                                                         │
│  Subscribed components re-render (via useSelector)           │
└──────────────────────────────────────────────────────────────┘
```

### Code Example (Redux Toolkit)

```typescript
// store/usersSlice.ts
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUsers = createAsyncThunk('users/fetchAll', async () => {
  const response = await fetch('/api/users');
  return response.json();
});

const usersSlice = createSlice({
  name: 'users',
  initialState: { items: [], loading: false, error: null } as UsersState,
  reducers: {
    addUser: (state, action) => {
      state.items.push(action.payload);
    },
    removeUser: (state, action) => {
      state.items = state.items.filter(u => u.id !== action.payload);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => { state.loading = true; })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message!;
      });
  },
});

// Component usage
function UsersList() {
  const dispatch = useAppDispatch();
  const { items, loading } = useAppSelector(state => state.users);

  useEffect(() => { dispatch(fetchUsers()); }, [dispatch]);

  return loading ? <Spinner /> : <ul>{items.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

**When to use Redux:**
- Large apps with complex shared state
- State needs to be accessible across many unrelated components
- Need time-travel debugging
- State transitions are complex (use reducers)

**When NOT to use Redux:**
- Server state (use React Query / SWR instead)
- Simple local UI state (use useState)
- Small apps (Context API is enough)

---

## Q34: What is the Virtual DOM and how does reconciliation work?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                 REACT RECONCILIATION                         │
│                                                              │
│  State change triggers re-render                             │
│                                                              │
│  Previous VDOM          New VDOM                            │
│  <ul>                   <ul>                                │
│    <li key="1">A  ──→     <li key="1">A  (no change)       │
│    <li key="2">B  ──→     <li key="2">C  (text changed)    │
│    <li key="3">C  ──→     <li key="4">D  (new element)     │
│  </ul>                  </ul>                                │
│                                                              │
│  React's diff algorithm (O(n)):                             │
│  1. Same type + same key → update props                     │
│  2. Different type → unmount old, mount new                 │
│  3. Keys help React match list items across renders         │
│                                                              │
│  Only real DOM changes are applied (batch update)           │
└──────────────────────────────────────────────────────────────┘
```

### Answer

The Virtual DOM is a lightweight JavaScript object representation of the real DOM. On every state change:
1. React creates a new VDOM tree
2. Diffs it against the previous VDOM (reconciliation)
3. Computes minimal set of real DOM operations needed
4. Applies them in a single batch

**Why keys matter in lists:**
```tsx
// Bad — React can't efficiently diff without keys
{users.map(u => <UserCard user={u} />)}

// Good — React can match items by ID across renders
{users.map(u => <UserCard key={u.id} user={u} />)}

// Never use index as key if list can reorder/filter
{users.map((u, i) => <UserCard key={i} user={u} />)} // BAD
```

---

## Q35: What is the difference between controlled and uncontrolled components?

### Code Example

```typescript
// CONTROLLED — React owns the state
function ControlledForm() {
  const [email, setEmail] = useState('');

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    // email is always in sync — validate, submit, etc.
    console.log(email);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}           // React controls value
        onChange={e => setEmail(e.target.value)}
      />
    </form>
  );
}

// UNCONTROLLED — DOM owns the state, accessed via ref
function UncontrolledForm() {
  const emailRef = useRef<HTMLInputElement>(null);

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    console.log(emailRef.current?.value); // read from DOM when needed
  }

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" defaultValue="" ref={emailRef} />
    </form>
  );
}
```

**Use controlled for:** validation, conditional UI based on input, real-time feedback.
**Use uncontrolled for:** file inputs, integrating with non-React libraries, simple forms where you only need value on submit.

---

## Q36: How do you optimize React performance?

### Answer

```
┌──────────────────────────────────────────────────────────────┐
│               REACT PERFORMANCE TOOLKIT                      │
│                                                              │
│  Prevent unnecessary re-renders:                             │
│  ├── React.memo    — memoize component (pure)               │
│  ├── useMemo       — memoize computed value                 │
│  └── useCallback   — stable function reference              │
│                                                              │
│  Reduce bundle size:                                         │
│  ├── Code splitting   — React.lazy + Suspense               │
│  ├── Tree shaking     — import named exports                │
│  └── Dynamic imports  — load heavy libs on demand           │
│                                                              │
│  List performance:                                           │
│  ├── Use keys         — help reconciliation                  │
│  └── Virtualization   — react-window (only render visible)   │
│                                                              │
│  State management:                                           │
│  ├── Co-locate state  — keep state close to where used      │
│  └── Avoid prop drilling — Context or state manager         │
└──────────────────────────────────────────────────────────────┘
```

```typescript
// 1. React.memo — skip re-render if props unchanged
const UserCard = React.memo(({ user }: { user: User }) => (
  <div>{user.name}</div>
));

// 2. Code splitting with lazy + Suspense
const HeavyChart = React.lazy(() => import('./HeavyChart'));

function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  );
}

// 3. Virtualized list — only renders visible items
import { FixedSizeList } from 'react-window';

function UsersList({ users }: { users: User[] }) {
  return (
    <FixedSizeList height={600} itemCount={users.length} itemSize={50} width="100%">
      {({ index, style }) => (
        <div style={style}>{users[index].name}</div>
      )}
    </FixedSizeList>
  );
}
```

---

## Q37: What is Context API and when should you use it vs Redux?

### Code Example

```typescript
// 1. Create context
interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | null>(null);

// 2. Provider — wraps the tree
export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = useCallback(async (email: string, password: string) => {
    const user = await authApi.login(email, password);
    setUser(user);
  }, []);

  const logout = useCallback(() => setUser(null), []);

  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// 3. Custom hook for safe consumption
export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
}

// 4. Usage
function Header() {
  const { user, logout } = useAuth();
  return <div>{user?.name} <button onClick={logout}>Logout</button></div>;
}
```

**Context vs Redux:**
| | Context API | Redux Toolkit |
|---|---|---|
| Setup | Minimal | More boilerplate |
| Performance | Re-renders all consumers on change | Selective re-renders via selectors |
| DevTools | No | Yes (time-travel) |
| Async | Manual | createAsyncThunk |
| Best for | Auth, theme, locale | Large shared state, complex async |

---

## Q38: What are React rules of Hooks?

### Answer

1. **Only call Hooks at the top level** — never inside loops, conditions, or nested functions
2. **Only call Hooks from React functions** — not regular JS functions

```typescript
// WRONG — conditional hook call
function UserProfile({ userId, isAdmin }) {
  if (isAdmin) {
    const [adminData, setAdminData] = useState(null); // BREAKS RULES
  }
  // ...
}

// CORRECT — condition inside the hook effect
function UserProfile({ userId, isAdmin }) {
  const [adminData, setAdminData] = useState(null);

  useEffect(() => {
    if (isAdmin) {
      fetchAdminData().then(setAdminData);
    }
  }, [isAdmin]);
}
```

Why: React tracks hooks by call order. If hooks are called conditionally, the order can change between renders, breaking the internal mapping.

---

## Q39: How do you handle forms in React with validation?

### Code Example (React Hook Form)

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
  confirmPassword: z.string(),
}).refine(d => d.password === d.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword'],
});

type FormData = z.infer<typeof schema>;

function RegisterForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
    setError,
  } = useForm<FormData>({ resolver: zodResolver(schema) });

  async function onSubmit(data: FormData) {
    try {
      await authApi.register(data);
    } catch (err) {
      setError('email', { message: 'Email already in use' });
    }
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} type="email" />
      {errors.email && <span>{errors.email.message}</span>}

      <input {...register('password')} type="password" />
      {errors.password && <span>{errors.password.message}</span>}

      <input {...register('confirmPassword')} type="password" />
      {errors.confirmPassword && <span>{errors.confirmPassword.message}</span>}

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
}
```

---

# Section 5: Next.js

---

## Q40: What is the difference between SSR, SSG, and ISR in Next.js?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│          NEXT.JS RENDERING STRATEGIES                        │
│                                                              │
│  SSG (Static Site Generation)                                │
│  Build time → generates HTML files → served from CDN        │
│  Use: blog posts, marketing pages, docs                      │
│  ├── Fast (CDN)   ✓                                         │
│  └── Stale data if not rebuilt   ✗                          │
│                                                              │
│  SSR (Server-Side Rendering)                                 │
│  Each request → server renders HTML → sends to client        │
│  Use: personalized pages, real-time data, SEO + dynamic      │
│  ├── Always fresh   ✓                                        │
│  └── Slower (server work per request)   ✗                   │
│                                                              │
│  ISR (Incremental Static Regeneration)                       │
│  Like SSG + auto-revalidation after N seconds               │
│  Use: product pages, news — needs SEO + periodic freshness   │
│  ├── Fast (CDN) + fresh enough   ✓                          │
│  └── Stale for up to revalidate seconds   ~                 │
│                                                              │
│  CSR (Client-Side Rendering)                                 │
│  HTML shell → JS fetches data in browser                     │
│  Use: dashboards, auth-gated pages (no SEO needed)          │
└──────────────────────────────────────────────────────────────┘
```

### Code Example (App Router)

```typescript
// SSG — generateStaticParams + no cache option
export async function generateStaticParams() {
  const products = await db.products.findMany({ select: { id: true } });
  return products.map(p => ({ id: p.id }));
}

export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await db.products.findUnique({ where: { id: params.id } });
  return <ProductView product={product} />;
}

// ISR — revalidate every 60 seconds
export const revalidate = 60;

// SSR — no caching (dynamic = 'force-dynamic' or no-store fetch)
export const dynamic = 'force-dynamic';

export default async function DashboardPage() {
  const data = await fetch('/api/stats', { cache: 'no-store' }).then(r => r.json());
  return <Dashboard data={data} />;
}
```

---

## Q41: What is the App Router vs Pages Router in Next.js?

### Answer

| | Pages Router (`/pages`) | App Router (`/app`) |
|---|---|---|
| Introduced | Next.js 1-12 | Next.js 13 |
| Default | Old projects | New projects (recommended) |
| Server Components | No | Yes (default) |
| Layouts | `_app.tsx` + manual | Nested `layout.tsx` (auto) |
| Data fetching | `getServerSideProps`, `getStaticProps` | `async` component functions, `fetch` |
| Loading states | Manual | `loading.tsx` (auto) |
| Error handling | Manual | `error.tsx` (auto) |

```typescript
// App Router structure
app/
├── layout.tsx           ← root layout (wraps all pages)
├── page.tsx             ← homepage
├── loading.tsx          ← auto shown during page load
├── error.tsx            ← auto shown on error
├── not-found.tsx        ← auto shown for 404
└── dashboard/
    ├── layout.tsx       ← nested layout (only for /dashboard/*)
    ├── page.tsx         ← /dashboard
    └── settings/
        └── page.tsx     ← /dashboard/settings

// Root layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Providers>          {/* Client components for context/state */}
          <Header />
          <main>{children}</main>
          <Footer />
        </Providers>
      </body>
    </html>
  );
}
```

---

## Q42: What are Server Components in Next.js?

### Answer

Server Components run only on the server — they can directly access databases, file systems, and secrets without exposing them to the client bundle.

```typescript
// Server Component (default in App Router — no 'use client')
// Runs on SERVER — can use DB, env secrets, fs
async function UserProfile({ userId }: { userId: string }) {
  // Direct DB access — no API route needed
  const user = await db.users.findUnique({ where: { id: userId } });

  if (!user) notFound(); // triggers not-found.tsx

  return (
    <div>
      <h1>{user.name}</h1>
      {/* Pass data to Client Component */}
      <EditButton userId={user.id} />
    </div>
  );
}

// Client Component — handles interactivity
'use client';

import { useState } from 'react';

function EditButton({ userId }: { userId: string }) {
  const [editing, setEditing] = useState(false);
  return <button onClick={() => setEditing(true)}>Edit</button>;
}
```

**Rules:**
- Server Components cannot use `useState`, `useEffect`, browser APIs
- Client Components cannot directly access server resources
- Server Components CAN render Client Components, but not vice versa (for server-only things)

---

## Q43: How do you handle API routes in Next.js App Router?

### Code Example

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const page = Number(searchParams.get('page') ?? 1);

  const users = await db.users.findMany({
    skip: (page - 1) * 20,
    take: 20,
  });

  return NextResponse.json({ users, page });
}

export async function POST(request: NextRequest) {
  const body = await request.json();

  const result = CreateUserSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json({ error: result.error.flatten() }, { status: 422 });
  }

  const user = await db.users.create({ data: result.data });
  return NextResponse.json(user, { status: 201 });
}

// app/api/users/[id]/route.ts — dynamic route
export async function GET(request: NextRequest, { params }: { params: { id: string } }) {
  const user = await db.users.findUnique({ where: { id: params.id } });
  if (!user) return NextResponse.json({ error: 'Not found' }, { status: 404 });
  return NextResponse.json(user);
}

export async function PATCH(request: NextRequest, { params }: { params: { id: string } }) {
  const body = await request.json();
  const updated = await db.users.update({ where: { id: params.id }, data: body });
  return NextResponse.json(updated);
}
```

---

## Q44: How do you implement authentication in Next.js with NextAuth?

### Code Example

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { compare } from 'bcrypt';

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        const user = await db.users.findUnique({
          where: { email: credentials?.email },
        });
        if (!user || !(await compare(credentials!.password, user.passwordHash))) {
          return null; // returning null triggers CredentialsSignin error
        }
        return { id: user.id, email: user.email, role: user.role };
      },
    }),
  ],
  callbacks: {
    jwt({ token, user }) {
      if (user) token.role = (user as any).role;
      return token;
    },
    session({ session, token }) {
      session.user.role = token.role as string;
      return session;
    },
  },
  pages: { signIn: '/login' },
  session: { strategy: 'jwt' },
});

export { handler as GET, handler as POST };

// middleware.ts — protect routes at edge
import { withAuth } from 'next-auth/middleware';

export default withAuth({
  callbacks: {
    authorized({ req, token }) {
      if (req.nextUrl.pathname.startsWith('/admin')) {
        return token?.role === 'admin';
      }
      return !!token;
    },
  },
});

export const config = { matcher: ['/dashboard/:path*', '/admin/:path*'] };
```

---

## Q45: What are Next.js performance optimization techniques?

### Answer

```typescript
// 1. Image optimization — automatic WebP, lazy loading, size optimization
import Image from 'next/image';

<Image
  src="/hero.jpg"
  width={1200}
  height={600}
  alt="Hero"
  priority          // above-the-fold images: preload
  placeholder="blur"
  blurDataURL="..."
/>

// 2. Font optimization — zero layout shift, self-hosted
import { Inter } from 'next/font/google';

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
});

// 3. Script optimization
import Script from 'next/script';

<Script src="https://analytics.example.com/script.js" strategy="lazyOnload" />
// strategy: 'beforeInteractive' | 'afterInteractive' | 'lazyOnload'

// 4. Parallel data fetching in Server Components
async function Page() {
  // These run in PARALLEL — not sequentially
  const [user, products, stats] = await Promise.all([
    fetchUser(),
    fetchProducts(),
    fetchStats(),
  ]);
  return <Dashboard user={user} products={products} stats={stats} />;
}

// 5. Streaming with Suspense — show parts of page as they load
export default function Page() {
  return (
    <>
      <Header />      {/* instant */}
      <Suspense fallback={<ProductsSkeleton />}>
        <Products />  {/* streams in when ready */}
      </Suspense>
    </>
  );
}
```

---

# Section 6: MongoDB

---

## Q46: What is the difference between SQL and NoSQL databases?

### Answer

| | SQL (PostgreSQL) | NoSQL (MongoDB) |
|---|---|---|
| Schema | Fixed, defined upfront | Flexible, per-document |
| Relations | Foreign keys + JOINs | Embedded docs or references |
| Scaling | Vertical (mostly) | Horizontal (sharding) |
| Transactions | ACID | ACID in 4.0+ (multi-doc) |
| Query language | SQL | JSON-like MQL |
| Best for | Structured data, complex relations | Variable schema, high write throughput |

---

## Q47: Explain the MongoDB Aggregation Pipeline.

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│              MONGODB AGGREGATION PIPELINE                    │
│                                                              │
│  Collection: orders                                          │
│  [{ status:"paid", amount:100, userId:"A" }, ...]           │
│                                                              │
│       ▼  $match — filter (like WHERE)                        │
│  [{ status:"paid", ... }]                                    │
│                                                              │
│       ▼  $group — aggregate (like GROUP BY)                  │
│  [{ _id:"A", total: 300, count: 3 }]                        │
│                                                              │
│       ▼  $lookup — join another collection                   │
│  [{ _id:"A", total:300, user: { name:"Alice" } }]           │
│                                                              │
│       ▼  $project — shape output (like SELECT)               │
│  [{ userName:"Alice", totalSpent:300 }]                      │
│                                                              │
│       ▼  $sort + $limit                                      │
│  Top 10 spenders                                             │
└──────────────────────────────────────────────────────────────┘
```

### Code Example

```javascript
// Get top 5 customers by total spend in the last 30 days
const result = await Order.aggregate([
  // Stage 1: filter
  {
    $match: {
      status: 'paid',
      createdAt: { $gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000) },
    },
  },
  // Stage 2: group by userId
  {
    $group: {
      _id: '$userId',
      totalSpent: { $sum: '$amount' },
      orderCount: { $sum: 1 },
      avgOrder: { $avg: '$amount' },
    },
  },
  // Stage 3: join users collection
  {
    $lookup: {
      from: 'users',
      localField: '_id',
      foreignField: '_id',
      as: 'user',
    },
  },
  { $unwind: '$user' },
  // Stage 4: shape output
  {
    $project: {
      _id: 0,
      name: '$user.name',
      email: '$user.email',
      totalSpent: 1,
      orderCount: 1,
    },
  },
  // Stage 5: sort and limit
  { $sort: { totalSpent: -1 } },
  { $limit: 5 },
]);
```

---

## Q48: What are indexes in MongoDB?

### Answer

Indexes are data structures that speed up query execution. Without an index, MongoDB scans every document (collection scan).

```javascript
// Single field index
db.users.createIndex({ email: 1 });        // ascending
db.users.createIndex({ email: 1 }, { unique: true }); // unique constraint

// Compound index — order matters
// Covers queries on: (status), (status + createdAt), but NOT (createdAt) alone
db.orders.createIndex({ status: 1, createdAt: -1 });

// Text index — for full-text search
db.products.createIndex({ name: 'text', description: 'text' });
db.products.find({ $text: { $search: 'wireless headphones' } });

// Partial index — only index documents matching a condition
db.orders.createIndex(
  { userId: 1 },
  { partialFilterExpression: { status: 'pending' } }
  // Only indexes pending orders — smaller index, faster writes
);

// Explain query — check if index is being used
db.users.find({ email: 'alice@example.com' }).explain('executionStats');
// Look for: IXSCAN (good) vs COLLSCAN (bad — no index used)

// Mongoose model definition
const userSchema = new Schema({
  email: { type: String, required: true, unique: true, index: true },
  role: { type: String, enum: ['admin', 'user'], index: true },
  createdAt: { type: Date, default: Date.now },
});
userSchema.index({ role: 1, createdAt: -1 }); // compound
```

---

## Q49: How do you model relationships in MongoDB?

### Answer

Two strategies: **embedding** (denormalized) vs **referencing** (normalized).

```javascript
// EMBEDDING — store related data in same document
// Use when: data is always read together, small array, 1:1 or 1:few
const orderSchema = new Schema({
  userId: ObjectId,
  items: [{                    // embedded — no separate collection
    productId: ObjectId,
    name: String,              // denormalized for read performance
    price: Number,
    quantity: Number,
  }],
  total: Number,
  status: String,
});

// REFERENCING — store IDs, populate when needed
// Use when: data is queried independently, large/unbounded arrays, many-to-many
const postSchema = new Schema({
  title: String,
  authorId: { type: ObjectId, ref: 'User' },   // reference
  tagIds: [{ type: ObjectId, ref: 'Tag' }],     // array of references
});

// Populate — fills in referenced documents
const posts = await Post.find()
  .populate('authorId', 'name email')  // only fetch name + email
  .populate('tagIds', 'name');

// HYBRID — embed frequently-accessed fields, reference the rest
const commentSchema = new Schema({
  postId: ObjectId,
  author: {
    id: ObjectId,
    name: String,              // denormalized — fast display
    avatar: String,
  },
  body: String,
  createdAt: Date,
});
```

---

## Q50: How do you handle transactions in MongoDB?

### Code Example

```javascript
// Multi-document ACID transaction (requires replica set or Atlas)
const session = await mongoose.startSession();

try {
  await session.withTransaction(async () => {
    // Both operations succeed or both fail atomically
    await Account.updateOne(
      { _id: fromAccountId, balance: { $gte: amount } },
      { $inc: { balance: -amount } },
      { session }
    );

    await Account.updateOne(
      { _id: toAccountId },
      { $inc: { balance: amount } },
      { session }
    );

    await Transaction.create([{
      from: fromAccountId,
      to: toAccountId,
      amount,
      status: 'completed',
    }], { session });
  });
} catch (err) {
  // Transaction automatically aborted on error
  throw err;
} finally {
  await session.endSession();
}
```

---

## Q51: What are common MongoDB query operators?

### Code Example

```javascript
// Comparison
User.find({ age: { $gte: 18, $lte: 65 } });
User.find({ role: { $in: ['admin', 'moderator'] } });
User.find({ deletedAt: { $exists: false } });
User.find({ email: { $ne: null } });

// Logical
User.find({ $and: [{ role: 'admin' }, { active: true }] });
User.find({ $or: [{ email: 'a@b.com' }, { phone: '123' }] });

// Array
Post.find({ tags: { $all: ['nodejs', 'typescript'] } }); // all tags present
Post.find({ tags: { $elemMatch: { $eq: 'nodejs' } } });  // at least one match
Post.find({ 'comments.2': { $exists: true } });           // has at least 3 comments

// Update operators
User.updateOne({ _id: id }, {
  $set: { name: 'Alice' },           // set specific fields
  $unset: { tempToken: '' },         // remove field
  $inc: { loginCount: 1 },           // increment
  $push: { tags: 'nodejs' },         // add to array
  $pull: { tags: 'php' },            // remove from array
  $addToSet: { roles: 'admin' },     // add if not exists
});

// Pagination pattern
async function paginate(page: number, limit: number) {
  const skip = (page - 1) * limit;
  const [items, total] = await Promise.all([
    User.find().sort({ createdAt: -1 }).skip(skip).limit(limit).lean(),
    User.countDocuments(),
  ]);
  return { items, total, pages: Math.ceil(total / limit), page };
}
```

---

## Q52: What is a Replica Set in MongoDB?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                   MONGODB REPLICA SET                        │
│                                                              │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐    │
│  │  Primary    │────▶│ Secondary 1 │     │ Secondary 2 │    │
│  │  (writes)   │────▶│  (replica)  │     │  (replica)  │    │
│  └─────────────┘     └─────────────┘     └─────────────┘    │
│         │                    │                  │            │
│  Accepts reads+writes  Reads only          Election voter    │
│                                                              │
│  Failover: if primary dies, secondaries elect a new primary  │
│  (automatic, ~10-30 seconds)                                 │
│                                                              │
│  Read preference options:                                    │
│  primary        — always read from primary                   │
│  primaryPreferred — primary if available, else secondary     │
│  secondary      — always read from secondary (eventual)      │
│  nearest        — lowest latency node                        │
└──────────────────────────────────────────────────────────────┘
```

---

# Section 7: PostgreSQL

---

## Q53: What are ACID properties?

### Answer

| Property | Meaning | Example |
|---|---|---|
| **Atomicity** | All operations succeed or all fail | Transfer: debit + credit are one unit |
| **Consistency** | DB goes from one valid state to another | Can't have negative balance |
| **Isolation** | Concurrent transactions don't interfere | Two users booking last seat |
| **Durability** | Committed data survives crashes | Confirmed payment persists after crash |

```sql
-- ATOMIC transaction — both succeed or both roll back
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
-- If second UPDATE fails: ROLLBACK automatically undoes first UPDATE
```

---

## Q54: What is the difference between INNER JOIN, LEFT JOIN, and RIGHT JOIN?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    SQL JOIN TYPES                            │
│                                                              │
│  Users:  A, B, C         Orders: B (2 orders), C (1 order)  │
│                                                              │
│  INNER JOIN — only matching rows from BOTH tables           │
│  Result: B (2 rows), C (1 row)  [A has no orders — excluded]│
│                                                              │
│  LEFT JOIN — all left rows + matching right rows            │
│  Result: A (NULL orders), B (2 rows), C (1 row)            │
│  Use: "all users and their orders (if any)"                 │
│                                                              │
│  RIGHT JOIN — all right rows + matching left rows           │
│  Result: all orders + their users (if user deleted → NULL)  │
│                                                              │
│  FULL OUTER JOIN — all rows from both, NULL where no match  │
└──────────────────────────────────────────────────────────────┘
```

```sql
-- INNER JOIN: users with at least one order
SELECT u.name, o.id, o.amount
FROM users u
INNER JOIN orders o ON o.user_id = u.id;

-- LEFT JOIN: all users, show orders if they have any
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON o.user_id = u.id
GROUP BY u.id, u.name;

-- Self join: employees and their managers
SELECT e.name as employee, m.name as manager
FROM employees e
LEFT JOIN employees m ON m.id = e.manager_id;
```

---

## Q55: What are indexes in PostgreSQL?

### Code Example

```sql
-- B-tree index (default) — equality, range, ORDER BY
CREATE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);

-- Partial index — only index a subset of rows
CREATE INDEX idx_orders_pending ON orders(created_at)
WHERE status = 'pending';
-- Only indexes pending orders — smaller, faster for this filter

-- Composite index — covers multi-column queries
-- Order matters: covers (status), (status, created_at), NOT (created_at) alone
CREATE INDEX idx_orders_status_date ON orders(status, created_at DESC);

-- GIN index — for arrays, JSONB, full-text search
CREATE INDEX idx_products_tags ON products USING GIN(tags);
CREATE INDEX idx_docs_body ON documents USING GIN(to_tsvector('english', body));

-- Explain query plan
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending' AND created_at > NOW() - INTERVAL '7 days';
-- Look for: Index Scan (good) vs Seq Scan (no index used)

-- Find unused indexes
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0;
```

---

## Q56: What are database transactions and isolation levels in PostgreSQL?

### Answer

```sql
-- Isolation levels (from weakest to strongest):

-- READ COMMITTED (default) — sees committed data at statement start
-- Prevents: dirty reads  |  Allows: non-repeatable reads, phantom reads
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- REPEATABLE READ — same rows return same values within transaction
-- Prevents: dirty reads, non-repeatable reads  |  Allows: phantom reads
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- SERIALIZABLE — full isolation, transactions appear sequential
-- Prevents: all anomalies  |  Cost: more lock contention
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Practical example: booking system
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Check seat availability
SELECT available_seats FROM flights WHERE id = 42 FOR UPDATE; -- lock row

-- If available > 0, book it
UPDATE flights SET available_seats = available_seats - 1 WHERE id = 42;
INSERT INTO bookings (flight_id, user_id) VALUES (42, 5);

COMMIT;
```

---

## Q57: What are CTEs (Common Table Expressions) in PostgreSQL?

### Code Example

```sql
-- CTE — named subquery, improves readability
WITH monthly_revenue AS (
  SELECT
    DATE_TRUNC('month', created_at) AS month,
    SUM(amount) AS revenue
  FROM orders
  WHERE status = 'paid'
  GROUP BY 1
),
revenue_with_growth AS (
  SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
    ROUND(
      (revenue - LAG(revenue) OVER (ORDER BY month)) /
      NULLIF(LAG(revenue) OVER (ORDER BY month), 0) * 100, 2
    ) AS growth_pct
  FROM monthly_revenue
)
SELECT * FROM revenue_with_growth ORDER BY month DESC;

-- Recursive CTE — for hierarchical data (org chart, category tree)
WITH RECURSIVE category_tree AS (
  -- Base case: root categories
  SELECT id, name, parent_id, 0 AS depth
  FROM categories
  WHERE parent_id IS NULL

  UNION ALL

  -- Recursive case: children
  SELECT c.id, c.name, c.parent_id, ct.depth + 1
  FROM categories c
  JOIN category_tree ct ON ct.id = c.parent_id
)
SELECT * FROM category_tree ORDER BY depth, name;
```

---

## Q58: How do you optimize slow queries in PostgreSQL?

### Answer

```sql
-- Step 1: Find slow queries
SELECT query, mean_exec_time, calls, total_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Step 2: Explain analyze
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.name, COUNT(o.id)
FROM users u JOIN orders o ON o.user_id = u.id
WHERE o.status = 'paid'
GROUP BY u.id;

-- Common fixes:

-- Add missing index
CREATE INDEX CONCURRENTLY idx_orders_user_status ON orders(user_id, status);
-- CONCURRENTLY — doesn't lock table during creation

-- Avoid SELECT * — only fetch needed columns
SELECT id, name, email FROM users; -- not SELECT *

-- Use connection pooling (PgBouncer)
-- Direct connections: each costs ~5MB RAM
-- PgBouncer: 1000 app connections → 20 actual DB connections

-- Vacuum and analyze to update statistics
VACUUM ANALYZE orders;

-- Partition large tables by date
CREATE TABLE orders_2025 PARTITION OF orders
FOR VALUES FROM ('2025-01-01') TO ('2026-01-01');
```

---

## Q59: What is connection pooling in PostgreSQL?

### Code Example

```typescript
// Without pooling: each request opens/closes DB connection (slow, limited)
// With pooling: connections are reused from a pool

// TypeORM connection pool config
TypeOrmModule.forRoot({
  type: 'postgres',
  url: process.env.DATABASE_URL,
  poolSize: 20,                    // max connections in pool
  connectTimeoutMS: 3000,          // fail fast if can't connect
  extra: {
    max: 20,                       // pg pool max
    min: 5,                        // keep min alive
    idleTimeoutMillis: 30000,      // close idle after 30s
    connectionTimeoutMillis: 2000,
  },
});

// PgBouncer (external pooler) — recommended for high traffic
// app → PgBouncer (transaction mode) → PostgreSQL
// Transaction mode: connection returned to pool after each transaction
// Allows: 10,000 app connections with only 100 DB connections
```

---

## Q60: What are Window Functions in PostgreSQL?

### Code Example

```sql
-- Window functions compute across rows related to current row
-- Unlike GROUP BY, they don't collapse rows

-- Row number, rank within partition
SELECT
  name,
  department,
  salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank_in_dept,
  RANK() OVER (ORDER BY salary DESC) AS overall_rank,
  DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- Running total
SELECT
  created_at::date AS day,
  amount,
  SUM(amount) OVER (ORDER BY created_at) AS running_total
FROM orders;

-- Moving average
SELECT
  date,
  revenue,
  AVG(revenue) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS week_avg
FROM daily_revenue;

-- Lead/Lag — compare to next/previous row
SELECT
  month,
  revenue,
  LAG(revenue, 1) OVER (ORDER BY month) AS prev_month,
  revenue - LAG(revenue, 1) OVER (ORDER BY month) AS change
FROM monthly_revenue;
```

---

# Section 8: TypeORM

---

## Q61: What is TypeORM and how does it work with NestJS?

### Code Example

```typescript
// entity — maps to a DB table
@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  name: string;

  @Column({ select: false }) // never returned in queries by default
  passwordHash: string;

  @Column({ type: 'enum', enum: UserRole, default: UserRole.USER })
  role: UserRole;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @DeleteDateColumn()       // soft delete — sets deletedAt instead of removing
  deletedAt: Date;

  // Relations
  @OneToMany(() => Order, order => order.user)
  orders: Order[];
}

// Related entity
@Entity('orders')
export class Order {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column('decimal', { precision: 10, scale: 2 })
  amount: number;

  @ManyToOne(() => User, user => user.orders)
  @JoinColumn({ name: 'user_id' })
  user: User;

  @Column({ name: 'user_id' })
  userId: string;
}
```

---

## Q62: How do you define relationships in TypeORM?

### Code Example

```typescript
// ONE-TO-ONE: User → Profile
@Entity()
export class Profile {
  @OneToOne(() => User, user => user.profile)
  @JoinColumn()  // this side owns the foreign key
  user: User;
}

@Entity()
export class User {
  @OneToOne(() => Profile, profile => profile.user, { cascade: true })
  profile: Profile;
}

// ONE-TO-MANY / MANY-TO-ONE: User → Orders
@Entity()
export class User {
  @OneToMany(() => Order, order => order.user)
  orders: Order[];
}

@Entity()
export class Order {
  @ManyToOne(() => User, user => user.orders, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'user_id' })
  user: User;
}

// MANY-TO-MANY: Posts ↔ Tags
@Entity()
export class Post {
  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable() // creates junction table post_tags
  tags: Tag[];
}

@Entity()
export class Tag {
  @ManyToMany(() => Post, post => post.tags)
  posts: Post[];
}

// Loading relations
const userWithOrders = await userRepo.findOne({
  where: { id },
  relations: { orders: true },
});

// Eager loading via option
@OneToMany(() => Order, order => order.user, { eager: true })
orders: Order[];
```

---

## Q63: What are TypeORM Migrations?

### Code Example

```typescript
// Generate migration from entity changes
// npm run typeorm migration:generate -- -n AddUserRoleColumn

// Generated migration file
export class AddUserRoleColumn1700000000000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`
      ALTER TABLE "users"
      ADD COLUMN "role" VARCHAR NOT NULL DEFAULT 'user'
    `);

    await queryRunner.query(`
      CREATE INDEX "IDX_users_role" ON "users" ("role")
    `);
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.query(`DROP INDEX "IDX_users_role"`);
    await queryRunner.query(`ALTER TABLE "users" DROP COLUMN "role"`);
  }
}

// Run migrations (in production startup or CI)
// npm run typeorm migration:run

// Revert last migration
// npm run typeorm migration:revert
```

---

## Q64: What is the difference between find() and QueryBuilder in TypeORM?

### Code Example

```typescript
// find() — simple queries
const users = await userRepo.find({
  where: { role: 'admin', isActive: true },
  order: { createdAt: 'DESC' },
  take: 20,
  skip: 0,
  relations: { orders: true },
  select: { id: true, email: true, name: true },
});

// QueryBuilder — complex queries with JOINs, subqueries, aggregations
const result = await userRepo
  .createQueryBuilder('u')
  .select(['u.id', 'u.name', 'u.email'])
  .addSelect('COUNT(o.id)', 'orderCount')
  .addSelect('SUM(o.amount)', 'totalSpent')
  .leftJoin('u.orders', 'o', 'o.status = :status', { status: 'paid' })
  .where('u.isActive = :active', { active: true })
  .andWhere('u.createdAt > :date', { date: new Date('2024-01-01') })
  .groupBy('u.id')
  .having('COUNT(o.id) > :minOrders', { minOrders: 5 })
  .orderBy('totalSpent', 'DESC')
  .limit(10)
  .getRawAndEntities();
```

---

## Q65: How do you handle transactions in TypeORM?

### Code Example

```typescript
// Method 1: EntityManager transaction
await dataSource.transaction(async (manager) => {
  const from = await manager.findOneOrFail(Account, {
    where: { id: fromId },
    lock: { mode: 'pessimistic_write' }, // row lock
  });

  if (from.balance < amount) throw new Error('Insufficient funds');

  await manager.decrement(Account, { id: fromId }, 'balance', amount);
  await manager.increment(Account, { id: toId }, 'balance', amount);

  await manager.save(Transfer, { fromId, toId, amount });
});
// Auto-commits on success, auto-rollbacks on any thrown error

// Method 2: QueryRunner (manual control)
const qr = dataSource.createQueryRunner();
await qr.connect();
await qr.startTransaction();

try {
  await qr.manager.save(User, newUser);
  await qr.manager.save(Profile, { user: newUser });
  await qr.commitTransaction();
} catch (err) {
  await qr.rollbackTransaction();
  throw err;
} finally {
  await qr.release();
}
```

---

# Section 9: Redis

---

## Q66: What is Redis and what are its primary use cases?

### Answer

Redis is an in-memory data store used as cache, message broker, and database.

```
┌──────────────────────────────────────────────────────────────┐
│                   REDIS USE CASES                            │
│                                                              │
│  Caching          — store API responses, DB query results    │
│  Session storage  — JWT refresh tokens, user sessions        │
│  Rate limiting    — token bucket per user/IP                 │
│  Pub/Sub          — real-time notifications                  │
│  Job queues       — BullMQ uses Redis as storage             │
│  Leaderboards     — sorted sets for rankings                 │
│  Distributed lock — prevent duplicate processing             │
│  Feature flags    — fast key lookups                         │
└──────────────────────────────────────────────────────────────┘
```

---

## Q67: What are the different Redis data structures?

### Code Example

```typescript
import Redis from 'ioredis';
const redis = new Redis(process.env.REDIS_URL);

// STRING — simple key-value
await redis.set('user:123:name', 'Alice');
await redis.set('session:abc', JSON.stringify(sessionData), 'EX', 3600); // expires in 1h
await redis.get('user:123:name');

// HASH — object with multiple fields (efficient for partial updates)
await redis.hset('user:123', { name: 'Alice', email: 'a@b.com', role: 'admin' });
await redis.hget('user:123', 'email');
await redis.hgetall('user:123'); // returns { name, email, role }

// LIST — ordered list (queue/stack operations)
await redis.rpush('notifications:user:123', JSON.stringify(notification)); // push to tail
await redis.lpop('queue:emails'); // pop from head (FIFO queue)
await redis.lrange('notifications:user:123', 0, 9); // get first 10

// SET — unique values, set operations
await redis.sadd('online_users', 'user:123', 'user:456');
await redis.sismember('online_users', 'user:123'); // is member?
await redis.smembers('online_users'); // get all

// SORTED SET — items with score, auto-sorted
await redis.zadd('leaderboard', 1500, 'alice', 2300, 'bob', 900, 'carol');
await redis.zrevrange('leaderboard', 0, 9, 'WITHSCORES'); // top 10
await redis.zincrby('leaderboard', 100, 'alice'); // add 100 to Alice's score
await redis.zrank('leaderboard', 'alice'); // Alice's rank (0-indexed)
```

---

## Q68: How do you implement caching with Redis in Node.js?

### Code Example

```typescript
// Cache-aside pattern (most common)
@Injectable()
export class ProductsService {
  constructor(
    @InjectRepository(Product) private repo: Repository<Product>,
    @Inject('REDIS') private redis: Redis,
  ) {}

  async findById(id: string): Promise<Product> {
    const key = `product:${id}`;
    const ttl = 300; // 5 minutes

    // 1. Try cache first
    const cached = await this.redis.get(key);
    if (cached) return JSON.parse(cached);

    // 2. Cache miss — query DB
    const product = await this.repo.findOneBy({ id });
    if (!product) throw new NotFoundException();

    // 3. Store in cache
    await this.redis.set(key, JSON.stringify(product), 'EX', ttl);
    return product;
  }

  async update(id: string, dto: UpdateProductDto): Promise<Product> {
    const updated = await this.repo.save({ id, ...dto });

    // Invalidate cache on write
    await this.redis.del(`product:${id}`);
    return updated;
  }

  // Cache full list with short TTL
  async findAll(): Promise<Product[]> {
    const key = 'products:all';
    const cached = await this.redis.get(key);
    if (cached) return JSON.parse(cached);

    const products = await this.repo.find({ order: { name: 'ASC' } });
    await this.redis.set(key, JSON.stringify(products), 'EX', 60);
    return products;
  }
}
```

---

## Q69: How do you implement rate limiting with Redis?

### Code Example

```typescript
// Token bucket rate limiter
async function rateLimitCheck(
  redis: Redis,
  identifier: string,     // userId or IP
  limit: number,          // max requests
  windowSeconds: number,  // per window
): Promise<{ allowed: boolean; remaining: number; resetAt: number }> {
  const key = `ratelimit:${identifier}`;
  const now = Date.now();

  // Atomic operation using Lua script — prevents race conditions
  const script = `
    local key = KEYS[1]
    local limit = tonumber(ARGV[1])
    local window = tonumber(ARGV[2])
    local now = tonumber(ARGV[3])

    local count = redis.call('INCR', key)
    if count == 1 then
      redis.call('EXPIRE', key, window)
    end

    local ttl = redis.call('TTL', key)
    return {count, ttl}
  `;

  const [count, ttl] = await redis.eval(script, 1, key, limit, windowSeconds, now) as number[];

  return {
    allowed: count <= limit,
    remaining: Math.max(0, limit - count),
    resetAt: now + ttl * 1000,
  };
}

// Middleware usage
app.use(async (req, res, next) => {
  const { allowed, remaining, resetAt } = await rateLimitCheck(
    redis,
    req.ip,
    100,   // 100 requests
    60,    // per 60 seconds
  );

  res.set('X-RateLimit-Remaining', remaining.toString());
  res.set('X-RateLimit-Reset', resetAt.toString());

  if (!allowed) {
    return res.status(429).json({ error: 'Too many requests' });
  }
  next();
});
```

---

## Q70: How do you implement session storage with Redis?

### Code Example

```typescript
// Refresh token rotation pattern (secure JWT with Redis)
@Injectable()
export class AuthService {
  constructor(
    @Inject('REDIS') private redis: Redis,
    private jwtService: JwtService,
  ) {}

  async generateTokens(userId: string) {
    const accessToken = this.jwtService.sign(
      { sub: userId },
      { expiresIn: '15m' }  // short-lived
    );

    const refreshToken = crypto.randomUUID();

    // Store refresh token in Redis with 7-day TTL
    await this.redis.set(
      `refresh:${refreshToken}`,
      userId,
      'EX',
      7 * 24 * 60 * 60,
    );

    return { accessToken, refreshToken };
  }

  async refreshTokens(refreshToken: string) {
    const userId = await this.redis.get(`refresh:${refreshToken}`);
    if (!userId) throw new UnauthorizedException('Invalid refresh token');

    // Rotate — delete old token, issue new one
    await this.redis.del(`refresh:${refreshToken}`);
    return this.generateTokens(userId);
  }

  async logout(refreshToken: string) {
    await this.redis.del(`refresh:${refreshToken}`);
  }

  async logoutAll(userId: string) {
    // Scan and delete all refresh tokens for this user
    // Better: use a set to track tokens per user
    const keys = await this.redis.keys(`refresh:*`);
    // Filter those belonging to userId...
  }
}
```

---

# Section 10: Message Queues

---

## Q71: What is BullMQ and how does it work?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    BULLMQ ARCHITECTURE                       │
│                                                              │
│  Producer (any service)                                      │
│  queue.add('send-email', { to, subject, body })             │
│       │                                                      │
│       ▼                                                      │
│  Redis (job storage)                                         │
│  ├── waiting   — jobs waiting to be picked up               │
│  ├── active    — jobs currently being processed             │
│  ├── completed — finished jobs                              │
│  ├── failed    — failed after max retries                   │
│  └── delayed   — scheduled for future                       │
│       │                                                      │
│       ▼                                                      │
│  Worker (picks up jobs)                                      │
│  worker.process(async (job) => { ... })                     │
│                                                              │
│  Features: retries, backoff, rate limiting, cron, events    │
└──────────────────────────────────────────────────────────────┘
```

### Code Example

```typescript
import { Queue, Worker, QueueEvents } from 'bullmq';

const connection = { host: 'localhost', port: 6379 };

// Producer — add jobs to queue
const emailQueue = new Queue('emails', { connection });

await emailQueue.add(
  'welcome-email',
  { to: 'user@example.com', name: 'Alice' },
  {
    attempts: 3,             // retry up to 3 times on failure
    backoff: { type: 'exponential', delay: 2000 },  // 2s, 4s, 8s
    delay: 5000,             // start after 5 seconds
    removeOnComplete: 100,   // keep last 100 completed jobs
    removeOnFail: 50,
  },
);

// Scheduled job (cron)
await emailQueue.add(
  'daily-digest',
  {},
  { repeat: { cron: '0 9 * * *' } } // every day at 9am
);

// Worker — process jobs
const worker = new Worker(
  'emails',
  async (job) => {
    const { to, name } = job.data;
    await mailer.send({ to, subject: 'Welcome!', html: `<p>Hi ${name}</p>` });
    return { sentAt: new Date() }; // stored as job result
  },
  {
    connection,
    concurrency: 5,  // process 5 jobs in parallel
  },
);

worker.on('completed', (job, result) => console.log(`Job ${job.id} done`));
worker.on('failed', (job, err) => console.error(`Job ${job?.id} failed: ${err.message}`));
```

---

## Q72: What is Kafka and when would you use it over BullMQ?

### Answer

| | BullMQ | Kafka |
|---|---|---|
| Storage | Redis | Distributed log on disk |
| Message retention | Until processed | Configurable (days/weeks) |
| Replay | No | Yes — replay from any offset |
| Throughput | Thousands/s | Millions/s |
| Consumers | Single worker per job | Consumer groups (many readers) |
| Best for | Task queues, job scheduling | Event streaming, audit logs, analytics |
| Complexity | Low | High |

```typescript
// Kafka with kafkajs
import { Kafka } from 'kafkajs';

const kafka = new Kafka({ brokers: ['kafka:9092'] });

// Producer — publish event
const producer = kafka.producer();
await producer.connect();
await producer.send({
  topic: 'order.created',
  messages: [
    {
      key: order.id,              // same key → same partition → ordered
      value: JSON.stringify(order),
      headers: { 'event-type': 'order.created', version: '1' },
    },
  ],
});

// Consumer — subscribe to topic
const consumer = kafka.consumer({ groupId: 'notifications-service' });
await consumer.connect();
await consumer.subscribe({ topic: 'order.created', fromBeginning: false });

await consumer.run({
  eachMessage: async ({ topic, partition, message }) => {
    const order = JSON.parse(message.value!.toString());
    await notificationService.sendOrderConfirmation(order);
    // Kafka auto-commits offset after successful processing
  },
});
```

---

## Q73: How do you implement a job queue in NestJS with BullMQ?

### Code Example

```typescript
// Install: @nestjs/bullmq bullmq
// queue.module.ts
@Module({
  imports: [
    BullModule.registerQueue({ name: 'emails' }),
    BullModule.registerQueue({ name: 'notifications' }),
  ],
  providers: [EmailProcessor],
  exports: [BullModule],
})
export class QueueModule {}

// email.processor.ts
@Processor('emails')
export class EmailProcessor extends WorkerHost {
  @Process('welcome')
  async handleWelcome(job: Job<{ to: string; name: string }>) {
    await this.mailerService.sendWelcome(job.data);
  }

  @Process('password-reset')
  async handlePasswordReset(job: Job<{ to: string; token: string }>) {
    await this.mailerService.sendPasswordReset(job.data);
  }

  @OnWorkerEvent('failed')
  onFailed(job: Job, err: Error) {
    this.logger.error(`Job ${job.id} failed: ${err.message}`);
  }
}

// users.service.ts — add jobs from service
@Injectable()
export class UsersService {
  constructor(@InjectQueue('emails') private emailQueue: Queue) {}

  async createUser(dto: CreateUserDto) {
    const user = await this.userRepo.save(dto);

    // Fire-and-forget — don't await
    this.emailQueue.add('welcome', {
      to: user.email,
      name: user.name,
    });

    return user;
  }
}
```

---

# Section 11: Security

---

## Q74: How does JWT authentication work end-to-end?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    JWT AUTHENTICATION FLOW                   │
│                                                              │
│  1. Login                                                    │
│  Client: POST /auth/login { email, password }               │
│  Server: verify credentials → sign JWT → return             │
│                                                              │
│  JWT structure:                                              │
│  header.payload.signature                                    │
│  { alg, typ }  { sub, email, role, exp }  HMAC-SHA256       │
│                                                              │
│  2. Use token                                                │
│  Client: GET /api/profile                                    │
│          Authorization: Bearer <token>                       │
│  Server: verify signature → check exp → attach req.user     │
│                                                              │
│  3. Token refresh (access token short-lived 15m)            │
│  Client: POST /auth/refresh { refreshToken }                │
│  Server: verify refresh token → issue new access token      │
│                                                              │
│  Security considerations:                                    │
│  ├── Store access token in memory (not localStorage)        │
│  ├── Store refresh token in httpOnly cookie                 │
│  └── Rotate refresh tokens on use                           │
└──────────────────────────────────────────────────────────────┘
```

---

## Q75: What is OAuth 2.0 and how does it work?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                    OAUTH 2.0 FLOW                            │
│              (Authorization Code + PKCE)                     │
│                                                              │
│  User clicks "Login with Google"                            │
│       │                                                      │
│       ▼                                                      │
│  Your App → redirects to Google with:                       │
│    client_id, redirect_uri, scope, state, code_challenge    │
│       │                                                      │
│       ▼                                                      │
│  Google: User logs in + grants permission                   │
│       │                                                      │
│       ▼                                                      │
│  Google → redirects to your redirect_uri with:             │
│    authorization_code, state                                │
│       │                                                      │
│       ▼                                                      │
│  Your App → server-to-server call to Google:                │
│    code + code_verifier → access_token + refresh_token      │
│       │                                                      │
│       ▼                                                      │
│  Google API calls with access_token                         │
│  Create/find user in your DB → issue your own JWT           │
└──────────────────────────────────────────────────────────────┘
```

---

## Q76: What is Single Sign-On (SSO)?

### Answer

SSO lets a user authenticate once and access multiple applications without re-logging in.

```
┌──────────────────────────────────────────────────────────────┐
│                    SSO FLOW (SAML/OIDC)                      │
│                                                              │
│  User visits App A (not logged in)                          │
│       │                                                      │
│       ▼                                                      │
│  App A redirects to Identity Provider (IdP)                 │
│  e.g., Okta, Azure AD, Auth0, Google Workspace              │
│       │                                                      │
│       ▼                                                      │
│  User logs in once at IdP                                   │
│       │                                                      │
│       ▼                                                      │
│  IdP issues token → redirects to App A                     │
│  App A validates token → user logged in                     │
│                                                              │
│  User visits App B (same browser session)                   │
│       │                                                      │
│       ▼                                                      │
│  App B redirects to IdP                                     │
│  IdP: session cookie exists → auto-issue token              │
│  App B: user logged in without re-entering credentials      │
└──────────────────────────────────────────────────────────────┘
```

---

## Q77: What are common API security vulnerabilities?

### Answer

```typescript
// 1. SQL/NoSQL Injection
// VULNERABLE
const user = await db.query(`SELECT * FROM users WHERE email = '${email}'`);
// SAFE — parameterized query
const user = await db.query('SELECT * FROM users WHERE email = $1', [email]);

// MongoDB injection — never use user input as query key
// VULNERABLE
await User.findOne({ [req.body.key]: req.body.value });
// SAFE
await User.findOne({ email: req.body.email });

// 2. XSS — sanitize output, use CSP headers
import DOMPurify from 'dompurify';
const safe = DOMPurify.sanitize(userInput);
// Set headers: Content-Security-Policy, X-XSS-Protection

// 3. CSRF — use SameSite cookies + CSRF tokens
res.cookie('session', token, {
  httpOnly: true,    // not accessible via JS
  secure: true,      // HTTPS only
  sameSite: 'strict', // not sent on cross-site requests
});

// 4. Mass assignment — never spread req.body directly
// VULNERABLE
await User.create({ ...req.body }); // user could set isAdmin: true
// SAFE — whitelist fields
const { name, email } = req.body;
await User.create({ name, email });

// 5. Sensitive data exposure
// Never log: passwords, tokens, PII
// Mask in responses
const { passwordHash, ...safeUser } = user; // don't return hash
```

---

# Section 12: Docker and Kubernetes

---

## Q78: What is Docker and how does containerization work?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│               DOCKER ARCHITECTURE                            │
│                                                              │
│  Your machine                                                │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Docker Engine                                      │     │
│  │                                                     │     │
│  │  Container A          Container B                   │     │
│  │  ┌─────────────┐      ┌─────────────┐               │     │
│  │  │  App code   │      │  App code   │               │     │
│  │  │  Node 20    │      │  Python 3   │               │     │
│  │  │  npm deps   │      │  pip deps   │               │     │
│  │  └─────────────┘      └─────────────┘               │     │
│  │  Isolated process, own filesystem, own network      │     │
│  └─────────────────────────────────────────────────────┘     │
│  OS Kernel (shared)                                          │
└──────────────────────────────────────────────────────────────┘
```

### Dockerfile Example

```dockerfile
# Multi-stage build — small production image
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci                        # install ALL deps (including dev)
COPY . .
RUN npm run build                 # compile TypeScript → dist/

FROM node:20-alpine AS production
WORKDIR /app
ENV NODE_ENV=production

# Only copy production deps
COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=builder /app/dist ./dist

# Security: run as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q -O- http://localhost:3000/health || exit 1

CMD ["node", "dist/main.js"]
```

### docker-compose.yml

```yaml
version: '3.9'
services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://user:pass@db:5432/myapp
      REDIS_URL: redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

---

## Q79: What is Kubernetes and what problems does it solve?

### Diagram

```
┌──────────────────────────────────────────────────────────────┐
│               KUBERNETES CLUSTER                             │
│                                                              │
│  Control Plane                                               │
│  ├── API Server      — receives kubectl commands            │
│  ├── Scheduler       — assigns pods to nodes                │
│  ├── Controller Mgr  — maintains desired state              │
│  └── etcd            — stores cluster state                 │
│                                                              │
│  Worker Nodes                                                │
│  ┌─────────────────────────┐                                 │
│  │  Node 1                 │                                 │
│  │  ┌────────┐ ┌────────┐  │                                 │
│  │  │ Pod A  │ │ Pod B  │  │  Pod = 1+ containers           │
│  │  └────────┘ └────────┘  │                                 │
│  │  kubelet + kube-proxy   │                                 │
│  └─────────────────────────┘                                 │
│                                                              │
│  Key objects:                                                │
│  Deployment  — desired # of pod replicas                    │
│  Service     — stable network endpoint for pods             │
│  Ingress     — HTTP routing into cluster                    │
│  ConfigMap   — non-secret config                            │
│  Secret      — sensitive config (base64 encoded)            │
│  HPA         — auto-scale pods based on CPU/memory          │
└──────────────────────────────────────────────────────────────┘
```

### Kubernetes manifests

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: myrepo/api:v1.2.0
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: database-url
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: api
spec:
  selector:
    app: api
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP

---
# hpa.yaml — auto-scale
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

# Section 13: Testing

---

## Q80: How do you write unit tests with Jest for Node.js?

### Code Example

```typescript
// users.service.spec.ts
import { Test } from '@nestjs/testing';
import { UsersService } from './users.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { User } from './entities/user.entity';

describe('UsersService', () => {
  let service: UsersService;
  let mockRepo: jest.Mocked<Repository<User>>;

  beforeEach(async () => {
    mockRepo = {
      find: jest.fn(),
      findOne: jest.fn(),
      findOneBy: jest.fn(),
      create: jest.fn(),
      save: jest.fn(),
      delete: jest.fn(),
    } as any;

    const module = await Test.createTestingModule({
      providers: [
        UsersService,
        { provide: getRepositoryToken(User), useValue: mockRepo },
      ],
    }).compile();

    service = module.get(UsersService);
  });

  describe('findById', () => {
    it('returns user when found', async () => {
      const user = { id: '1', email: 'a@b.com', name: 'Alice' } as User;
      mockRepo.findOneBy.mockResolvedValue(user);

      const result = await service.findById('1');
      expect(result).toEqual(user);
      expect(mockRepo.findOneBy).toHaveBeenCalledWith({ id: '1' });
    });

    it('throws NotFoundException when user not found', async () => {
      mockRepo.findOneBy.mockResolvedValue(null);

      await expect(service.findById('999')).rejects.toThrow(NotFoundException);
    });
  });

  describe('create', () => {
    it('hashes password before saving', async () => {
      const dto = { email: 'b@c.com', name: 'Bob', password: 'password123' };
      const savedUser = { id: '2', ...dto, passwordHash: 'hashed' } as any;

      mockRepo.create.mockReturnValue(savedUser);
      mockRepo.save.mockResolvedValue(savedUser);

      const result = await service.create(dto);
      expect(result.passwordHash).not.toBe('password123'); // was hashed
      expect(result).not.toHaveProperty('password');
    });
  });
});
```

---

## Q81: How do you test NestJS controllers with supertest?

### Code Example

```typescript
// users.e2e-spec.ts
import { Test } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import request from 'supertest';
import { AppModule } from '../src/app.module';
import { getDataSource } from './test-utils';

describe('UsersController (e2e)', () => {
  let app: INestApplication;
  let authToken: string;

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = module.createNestApplication();
    app.useGlobalPipes(new ValidationPipe({ whitelist: true }));
    await app.init();

    // Login to get token
    const res = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'admin@test.com', password: 'password' });
    authToken = res.body.accessToken;
  });

  afterAll(async () => {
    await app.close();
  });

  it('GET /users — returns paginated users', async () => {
    const res = await request(app.getHttpServer())
      .get('/users')
      .set('Authorization', `Bearer ${authToken}`)
      .query({ page: 1, limit: 10 })
      .expect(200);

    expect(res.body).toMatchObject({
      data: expect.arrayContaining([
        expect.objectContaining({ id: expect.any(String), email: expect.any(String) })
      ]),
    });
  });

  it('POST /users — validates input', async () => {
    const res = await request(app.getHttpServer())
      .post('/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send({ email: 'not-an-email', password: '123' })
      .expect(422);

    expect(res.body.error.details).toHaveProperty('email');
    expect(res.body.error.details).toHaveProperty('password');
  });
});
```

---

## Q82: What is the difference between unit, integration, and E2E tests?

### Answer

| | Unit | Integration | E2E |
|---|---|---|---|
| Scope | Single function/class | Multiple units together | Full system |
| Dependencies | All mocked | Real DB/Redis in test env | Real system |
| Speed | Very fast (ms) | Medium (seconds) | Slow (minutes) |
| Isolation | High | Medium | Low |
| Confidence | Low (mocks can lie) | Medium | High |
| Example | Service method | Controller + Service + DB | Full HTTP request flow |

```
Test pyramid:
        /\
       /E2E\          few, slow, high confidence
      /──────\
     / Integr \       moderate
    /──────────\
   /   Unit     \     many, fast, low confidence
  ──────────────────
```

---

# Section 14: GraphQL

---

## Q83: What is GraphQL and how does it differ from REST?

### Code Example (NestJS + GraphQL)

```typescript
// schema-first or code-first (code-first shown)

// entity
@ObjectType()
export class User {
  @Field(() => ID)
  id: string;

  @Field()
  email: string;

  @Field()
  name: string;

  @Field(() => [Order])
  orders: Order[];
}

// resolver
@Resolver(() => User)
export class UsersResolver {
  constructor(private usersService: UsersService) {}

  @Query(() => [User], { name: 'users' })
  findAll() {
    return this.usersService.findAll();
  }

  @Query(() => User, { nullable: true })
  user(@Args('id', { type: () => ID }) id: string) {
    return this.usersService.findById(id);
  }

  @Mutation(() => User)
  @UseGuards(JwtAuthGuard)
  createUser(@Args('input') input: CreateUserInput) {
    return this.usersService.create(input);
  }

  // Field resolver — loads orders only when requested
  @ResolveField(() => [Order])
  orders(@Parent() user: User) {
    return this.ordersService.findByUserId(user.id);
  }
}
```

---

## Q84: What is the N+1 problem in GraphQL and how do you solve it?

### Code Example (DataLoader)

```typescript
// PROBLEM: resolving orders for 100 users = 100 separate DB queries
@ResolveField('orders')
async orders(@Parent() user: User) {
  return this.ordersService.findByUserId(user.id); // called 100 times!
}

// SOLUTION: DataLoader — batches all IDs into one query
import DataLoader from 'dataloader';

@Injectable()
export class OrdersLoader {
  constructor(private ordersService: OrdersService) {}

  // Called once with ALL user IDs gathered in one tick
  readonly loader = new DataLoader<string, Order[]>(async (userIds) => {
    const orders = await this.ordersService.findByUserIds([...userIds]);

    // Group by userId
    const grouped = new Map<string, Order[]>();
    for (const order of orders) {
      const list = grouped.get(order.userId) ?? [];
      list.push(order);
      grouped.set(order.userId, list);
    }

    return userIds.map(id => grouped.get(id) ?? []);
  });
}

// Resolver uses DataLoader
@ResolveField('orders')
orders(@Parent() user: User, @Context() ctx: GqlContext) {
  return ctx.ordersLoader.load(user.id); // batched automatically
}
```

---

# Section 15: TypeScript

---

## Q85: What are TypeScript Generics?

### Code Example

```typescript
// Generic function — works with any type
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}
first([1, 2, 3]);        // T = number
first(['a', 'b']);       // T = string

// Generic with constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// Generic interface — reusable pagination response
interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  pages: number;
}

async function getUsers(): Promise<PaginatedResponse<User>> { ... }
async function getOrders(): Promise<PaginatedResponse<Order>> { ... }

// Generic repository pattern
interface Repository<T, ID> {
  findById(id: ID): Promise<T | null>;
  findAll(options?: FindOptions): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: ID): Promise<void>;
}

class UserRepository implements Repository<User, string> { ... }
```

---

## Q86: What are TypeScript Utility Types?

### Code Example

```typescript
interface User {
  id: string;
  email: string;
  name: string;
  passwordHash: string;
  role: 'admin' | 'user';
  createdAt: Date;
}

// Partial — all fields optional (useful for update DTOs)
type UpdateUserDto = Partial<Pick<User, 'name' | 'email'>>;

// Pick — only selected fields
type UserPublic = Pick<User, 'id' | 'email' | 'name' | 'role'>;

// Omit — all fields except specified
type UserWithoutPassword = Omit<User, 'passwordHash'>;

// Required — all fields required
type UserRequired = Required<User>;

// Readonly — prevent mutation
type ImmutableUser = Readonly<User>;

// Record — key-value mapping
type RolePermissions = Record<'admin' | 'user' | 'viewer', string[]>;
const permissions: RolePermissions = {
  admin: ['read', 'write', 'delete'],
  user: ['read', 'write'],
  viewer: ['read'],
};

// ReturnType + Parameters — extract function types
type AsyncFn = () => Promise<User>;
type Resolved = Awaited<ReturnType<AsyncFn>>; // User
```

---

## Q87: What are TypeScript Decorators?

### Code Example (used in NestJS)

```typescript
// Class decorator
function Controller(path: string) {
  return function(target: Function) {
    Reflect.defineMetadata('path', path, target);
  };
}

// Method decorator
function Get(path: string) {
  return function(target: any, key: string, descriptor: PropertyDescriptor) {
    Reflect.defineMetadata('method', 'GET', target, key);
    Reflect.defineMetadata('path', path, target, key);
    return descriptor;
  };
}

// Parameter decorator
function Body() {
  return function(target: any, key: string, index: number) {
    Reflect.defineMetadata('body', index, target, key);
  };
}

// Property decorator — class-validator uses this
function IsEmail() {
  return function(target: any, propertyKey: string) {
    // registers validation rule for this property
    registerDecorator({ target: target.constructor, propertyName: propertyKey,
      validator: { validate: (value) => /\S+@\S+\.\S+/.test(value) }
    });
  };
}
```

---

## Q88: What are advanced TypeScript patterns?

### Code Example

```typescript
// Discriminated union — type-safe state machines
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function renderState<T>(state: RequestState<T>) {
  switch (state.status) {
    case 'idle':    return 'idle';
    case 'loading': return 'loading...';
    case 'success': return state.data;     // TypeScript knows data exists here
    case 'error':   return state.error;    // TypeScript knows error exists here
  }
}

// Conditional types
type IsArray<T> = T extends any[] ? true : false;
type ElementType<T> = T extends (infer E)[] ? E : never;
type ElementType<number[]> // = number

// Template literal types
type EventName = 'user' | 'order' | 'product';
type EventTypes = `${EventName}.created` | `${EventName}.updated` | `${EventName}.deleted`;
// = 'user.created' | 'user.updated' | ... | 'product.deleted'

// Builder pattern with TypeScript
class QueryBuilder<T> {
  private filters: Partial<T> = {};
  private _limit = 20;

  where(filter: Partial<T>): this {
    this.filters = { ...this.filters, ...filter };
    return this;
  }

  limit(n: number): this {
    this._limit = n;
    return this;
  }

  build() {
    return { filters: this.filters, limit: this._limit };
  }
}

new QueryBuilder<User>().where({ role: 'admin' }).limit(10).build();
```

---

## Q89: How do you implement Styled Components and Tailwind CSS?

### Code Example

```typescript
// Styled Components — CSS-in-JS
import styled, { css } from 'styled-components';

const Button = styled.button<{ variant: 'primary' | 'secondary'; size?: 'sm' | 'md' }>`
  padding: ${p => p.size === 'sm' ? '4px 8px' : '8px 16px'};
  border-radius: 4px;
  cursor: pointer;
  font-weight: 600;

  ${p => p.variant === 'primary' && css`
    background: #3b82f6;
    color: white;
    &:hover { background: #2563eb; }
  `}

  ${p => p.variant === 'secondary' && css`
    background: transparent;
    border: 1px solid #3b82f6;
    color: #3b82f6;
  `}
`;

// Usage
<Button variant="primary" size="sm" onClick={handleClick}>Save</Button>

// Tailwind CSS — utility classes
function Card({ title, children }: { title: string; children: React.ReactNode }) {
  return (
    <div className="rounded-lg shadow-md p-6 bg-white dark:bg-gray-800">
      <h2 className="text-xl font-bold text-gray-900 dark:text-white mb-4">{title}</h2>
      <div className="text-gray-600 dark:text-gray-300">{children}</div>
    </div>
  );
}

// Material UI
import { Button, TextField, Box } from '@mui/material';

function LoginForm() {
  return (
    <Box component="form" sx={{ display: 'flex', flexDirection: 'column', gap: 2 }}>
      <TextField label="Email" type="email" variant="outlined" fullWidth />
      <TextField label="Password" type="password" variant="outlined" fullWidth />
      <Button variant="contained" size="large" type="submit">Login</Button>
    </Box>
  );
}
```

---

## Q90: How do you implement responsive design with CSS?

### Code Example

```css
/* Mobile-first approach */
.container {
  width: 100%;
  padding: 0 16px;
}

/* Tablet */
@media (min-width: 768px) {
  .container {
    max-width: 768px;
    margin: 0 auto;
    padding: 0 24px;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .container {
    max-width: 1200px;
    padding: 0 32px;
  }
}

/* CSS Grid responsive layout — no media query needed */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 24px;
}

/* Flexbox responsive nav */
.nav {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
```

---

## Q91: How do you write non-blocking code and use multi-threading in Node.js?

### Code Example

```typescript
// Non-blocking patterns

// 1. Never block the event loop with CPU-intensive work
// WRONG — blocks all other requests
app.get('/hash', (req, res) => {
  const hash = crypto.scryptSync(req.body.password, 'salt', 64); // BLOCKING
  res.send(hash.toString('hex'));
});

// CORRECT — async version
app.get('/hash', async (req, res) => {
  const hash = await new Promise<Buffer>((resolve, reject) => {
    crypto.scrypt(req.body.password, 'salt', 64, (err, key) => {
      if (err) reject(err); else resolve(key);
    });
  });
  res.send(hash.toString('hex'));
});

// 2. Worker thread for CPU-intensive tasks
import { Worker, isMainThread, parentPort, workerData } from 'worker_threads';

// worker.ts
if (!isMainThread) {
  const result = performHeavyComputation(workerData);
  parentPort!.postMessage(result);
}

// main service
function runInWorker(data: any): Promise<any> {
  return new Promise((resolve, reject) => {
    const worker = new Worker(__filename, { workerData: data });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}

// 3. Parallel async operations
async function loadDashboard(userId: string) {
  // Sequential — slow: total = a + b + c time
  // const user = await getUser(userId);
  // const orders = await getOrders(userId);
  // const stats = await getStats(userId);

  // Parallel — fast: total = max(a, b, c) time
  const [user, orders, stats] = await Promise.all([
    getUser(userId),
    getOrders(userId),
    getStats(userId),
  ]);

  return { user, orders, stats };
}
```

---

## Q92: How do you implement WebSockets in Node.js/NestJS?

### Code Example

```typescript
// NestJS WebSocket Gateway
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
  OnGatewayConnection,
  OnGatewayDisconnect,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: { origin: process.env.CLIENT_URL },
  namespace: '/notifications',
})
export class NotificationsGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  private userSocketMap = new Map<string, string>(); // userId → socketId

  async handleConnection(socket: Socket) {
    const token = socket.handshake.auth.token;
    try {
      const payload = jwt.verify(token, process.env.JWT_SECRET!);
      this.userSocketMap.set(payload.sub, socket.id);
      socket.join(`user:${payload.sub}`); // room per user
    } catch {
      socket.disconnect();
    }
  }

  handleDisconnect(socket: Socket) {
    for (const [userId, socketId] of this.userSocketMap) {
      if (socketId === socket.id) {
        this.userSocketMap.delete(userId);
        break;
      }
    }
  }

  // Send notification to specific user
  sendToUser(userId: string, event: string, data: any) {
    this.server.to(`user:${userId}`).emit(event, data);
  }

  @SubscribeMessage('message')
  handleMessage(@MessageBody() data: any, @ConnectedSocket() client: Socket) {
    // broadcast to room
    client.broadcast.to(data.room).emit('message', data);
  }
}

// Emit from any service
@Injectable()
export class OrdersService {
  constructor(private gateway: NotificationsGateway) {}

  async createOrder(dto: CreateOrderDto) {
    const order = await this.orderRepo.save(dto);
    // Push real-time notification to user
    this.gateway.sendToUser(dto.userId, 'order:created', order);
    return order;
  }
}
```

---

## Q93: How do you implement file uploads in Node.js?

### Code Example

```typescript
// NestJS file upload with Multer
import { FileInterceptor, FilesInterceptor } from '@nestjs/platform-express';
import { diskStorage, memoryStorage } from 'multer';

@Controller('uploads')
export class UploadsController {
  // Single file
  @Post('avatar')
  @UseGuards(JwtAuthGuard)
  @UseInterceptors(FileInterceptor('file', {
    storage: memoryStorage(), // keep in memory for S3 upload
    fileFilter: (req, file, cb) => {
      if (!file.mimetype.match(/^image\/(jpeg|png|webp)$/)) {
        return cb(new BadRequestException('Only JPEG/PNG/WebP allowed'), false);
      }
      cb(null, true);
    },
    limits: { fileSize: 2 * 1024 * 1024 }, // 2MB max
  }))
  async uploadAvatar(
    @UploadedFile() file: Express.Multer.File,
    @CurrentUser() user: User,
  ) {
    // Upload to S3
    const key = `avatars/${user.id}/${Date.now()}-${file.originalname}`;
    await s3.putObject({
      Bucket: process.env.S3_BUCKET!,
      Key: key,
      Body: file.buffer,
      ContentType: file.mimetype,
    });

    const url = `https://${process.env.S3_BUCKET}.s3.amazonaws.com/${key}`;
    await this.usersService.updateAvatar(user.id, url);
    return { url };
  }
}
```

---

## Q94: How do you implement pagination in a REST API?

### Code Example

```typescript
// Cursor-based pagination (better for large datasets)
@Get()
async findAll(@Query() query: PaginationQueryDto) {
  const { cursor, limit = 20, sortBy = 'createdAt', order = 'DESC' } = query;

  const qb = this.userRepo.createQueryBuilder('u').orderBy(`u.${sortBy}`, order).take(limit + 1);

  if (cursor) {
    const decoded = Buffer.from(cursor, 'base64').toString('utf8');
    const { id, value } = JSON.parse(decoded);
    qb.where(`(u.${sortBy}, u.id) < (:value, :id)`, { value, id });
  }

  const items = await qb.getMany();
  const hasNext = items.length > limit;
  if (hasNext) items.pop();

  const nextCursor = hasNext
    ? Buffer.from(JSON.stringify({
        id: items.at(-1)!.id,
        value: items.at(-1)![sortBy],
      })).toString('base64')
    : null;

  return { items, nextCursor, hasNext };
}

// Offset pagination (simpler, good for small datasets)
@Get()
async findAll(@Query('page') page = 1, @Query('limit') limit = 20) {
  const [items, total] = await this.userRepo.findAndCount({
    order: { createdAt: 'DESC' },
    take: Math.min(limit, 100), // cap at 100
    skip: (page - 1) * limit,
  });

  return {
    data: items,
    meta: { total, page, pages: Math.ceil(total / limit), limit },
  };
}
```

---

## Q95: What is the 3Scale API Gateway and how is it used?

### Answer

3Scale is Red Hat's API management platform. It sits in front of your services and provides:

```
┌──────────────────────────────────────────────────────────────┐
│                   3SCALE API GATEWAY                         │
│                                                              │
│  Client                                                      │
│    │  API call + API key or OAuth token                     │
│    ▼                                                         │
│  3Scale Gateway                                              │
│  ├── Authentication  — validate API key / OAuth token       │
│  ├── Authorization   — check plan limits and permissions    │
│  ├── Rate Limiting   — enforce calls/minute by plan         │
│  ├── Analytics       — log every call for billing/reporting │
│  └── Routing         — forward to backend service           │
│    ▼                                                         │
│  Your Backend Service                                        │
│                                                              │
│  Concepts:                                                   │
│  Account   — company/developer using the API                │
│  App       — an application with an API key                 │
│  Plan       — set of rate limits (free: 100/day, pro: 10k)  │
│  Metrics   — track specific endpoints separately            │
└──────────────────────────────────────────────────────────────┘
```

---

## Q96: How do you implement health checks in Node.js?

### Code Example

```typescript
// NestJS Terminus health checks
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private redis: MicroserviceHealthIndicator,
    private http: HttpHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  async check() {
    return this.health.check([
      // DB health
      () => this.db.pingCheck('database'),

      // Redis health
      () => this.redis.pingCheck('redis', {
        transport: Transport.REDIS,
        options: { host: 'redis', port: 6379 },
      }),

      // External dependency
      () => this.http.pingCheck('payment-service', 'https://payments.internal/health'),

      // Custom check — disk space
      () => this.disk.checkStorage('storage', { path: '/', thresholdPercent: 0.9 }),

      // Custom check — memory
      () => this.memory.checkHeap('memory_heap', 300 * 1024 * 1024), // 300MB
    ]);
  }
}

// Kubernetes uses this for liveness/readiness probes
// livenessProbe: is the process alive? restart if fails
// readinessProbe: is the process ready to receive traffic? remove from LB if fails
```

---

## Q97: How do you implement logging in a Node.js production application?

### Code Example

```typescript
// Pino — fast structured JSON logger
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: { service: 'api', version: process.env.APP_VERSION },
  timestamp: pino.stdTimeFunctions.isoTime,
  // In production: output as JSON, pipe to log aggregator (Datadog, CloudWatch)
  // In development: pretty print
  transport: process.env.NODE_ENV === 'development'
    ? { target: 'pino-pretty', options: { colorize: true } }
    : undefined,
});

// Request logging middleware
app.use((req, res, next) => {
  const reqLogger = logger.child({
    requestId: req.headers['x-request-id'] ?? crypto.randomUUID(),
    method: req.method,
    url: req.url,
  });

  req.logger = reqLogger; // attach to request for downstream use

  const start = Date.now();
  res.on('finish', () => {
    reqLogger.info({
      statusCode: res.statusCode,
      durationMs: Date.now() - start,
    }, 'request completed');
  });

  next();
});

// Use in services
function createUser(dto: CreateUserDto) {
  logger.info({ email: dto.email }, 'creating user'); // structure, not concatenation
  // NOT: logger.info(`creating user ${dto.email}`) — can't search structured logs
}
```

---

## Q98: How do you implement search functionality in a Node.js backend?

### Code Example

```typescript
// Option 1: PostgreSQL full-text search
const results = await dataSource.query(`
  SELECT id, name, description,
    ts_rank(search_vector, query) AS rank
  FROM products,
    to_tsquery('english', $1) AS query
  WHERE search_vector @@ query
  ORDER BY rank DESC
  LIMIT 20
`, [searchTerm.split(' ').join(' & ')]);

// Option 2: MongoDB text index
await Product.find(
  { $text: { $search: searchTerm } },
  { score: { $meta: 'textScore' } }
).sort({ score: { $meta: 'textScore' } }).limit(20);

// Option 3: Elasticsearch (production-grade)
const { hits } = await esClient.search({
  index: 'products',
  body: {
    query: {
      multi_match: {
        query: searchTerm,
        fields: ['name^3', 'description', 'tags'], // name weighted 3x
        fuzziness: 'AUTO', // typo-tolerant
      },
    },
    highlight: {
      fields: { name: {}, description: {} },
    },
    from: (page - 1) * 20,
    size: 20,
  },
});
```

---

## Q99: How do you implement CI/CD for a Node.js application?

### GitHub Actions Example

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      redis:
        image: redis:7
        options: --health-cmd "redis-cli ping" --health-interval 10s

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - run: npm ci
      - run: npm run lint
      - run: npm run build
      - run: npm test -- --coverage
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-secret-at-least-32-characters-long

      - uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v4

      - name: Build and push Docker image
        run: |
          docker build -t ${{ secrets.REGISTRY }}/api:${{ github.sha }} .
          docker push ${{ secrets.REGISTRY }}/api:${{ github.sha }}

      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/api \
            api=${{ secrets.REGISTRY }}/api:${{ github.sha }}
          kubectl rollout status deployment/api
```

---

## Q100: What are the SOLID principles and how do they apply in Node.js/NestJS?

### Answer

```typescript
// S — Single Responsibility: one class, one reason to change
// BAD: UserService does auth + email + DB
class UserService {
  async register(dto: CreateUserDto) {
    const user = await this.db.save(dto);    // DB concern
    await this.mailer.send(user.email);       // email concern
    return jwt.sign({ sub: user.id }, secret); // auth concern
  }
}

// GOOD: separate responsibilities
class UserRepository { save(dto) { ... } }
class AuthService { generateToken(user) { ... } }
class MailerService { sendWelcome(email) { ... } }
class UserRegistrationService {
  constructor(private userRepo, private auth, private mailer) {}
  async register(dto) {
    const user = await this.userRepo.save(dto);
    await this.mailer.sendWelcome(user.email);
    return this.auth.generateToken(user);
  }
}

// O — Open/Closed: open for extension, closed for modification
// Add new payment providers without modifying existing code
interface PaymentProvider { charge(amount: number): Promise<string>; }
class StripeProvider implements PaymentProvider { ... }
class PaypalProvider implements PaymentProvider { ... }

// L — Liskov Substitution: subclass can replace parent
// I — Interface Segregation: many small interfaces over one large
interface Readable { find(): Promise<any[]>; }
interface Writable { save(data: any): Promise<any>; }
// ReadOnlyRepository only implements Readable

// D — Dependency Inversion: depend on abstractions (interfaces), not implementations
// NestJS DI does this: inject the interface, not the concrete class
@Injectable()
class UsersService {
  constructor(
    @Inject(USER_REPOSITORY) // inject the interface token
    private userRepo: IUserRepository,
  ) {}
}
```

---

# Quick Reference: Key Concepts Summary

| Topic | Most Asked |
|---|---|
| Node.js | Event loop, non-blocking I/O, streams, clustering |
| Express | Middleware, error handling, auth, validation |
| NestJS | Modules, Guards, Pipes, Interceptors, DI |
| React | Hooks, Virtual DOM, Redux, performance |
| Next.js | SSR vs SSG vs ISR, App Router, Server Components |
| MongoDB | Aggregation pipeline, indexes, transactions |
| PostgreSQL | ACID, JOINs, indexes, CTEs, window functions |
| TypeORM | Entities, relations, migrations, QueryBuilder |
| Redis | Data structures, caching, rate limiting |
| Queues | BullMQ jobs, Kafka streams |
| Security | JWT, OAuth, XSS, injection prevention |
| Docker | Multi-stage Dockerfile, docker-compose |
| Kubernetes | Deployment, Service, HPA |
| Testing | Jest unit tests, supertest E2E |
| TypeScript | Generics, utility types, decorators |


