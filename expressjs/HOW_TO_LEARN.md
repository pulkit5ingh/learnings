# HOW TO LEARN — Backend Developer using Express.js

A practical guide to getting the most out of this curriculum.

---

## The Core Principle

Reading about code does not make you a programmer. Every concept in this curriculum must be typed, run, broken, and fixed by you. If you haven't written the code yourself, you haven't learned it.

---

## Your Daily Workflow

### Before You Start an Assignment
1. Read the entire assignment file first. Understand what you're building.
2. Open a fresh project directory (e.g., `mkdir a1-1-node-core && cd a1-1-node-core && npm init -y`).
3. Do not copy-paste code blocks. Type them. Your muscle memory is part of learning.

### While Working Through an Assignment
1. Run the code after every meaningful addition. Don't write 50 lines and then run.
2. When something doesn't work, read the error message carefully before Googling.
3. Deliberately break things — remove a middleware, swap the order of functions, delete an import — then fix them. This builds intuition.
4. Add `console.log` statements to inspect values before you're confident about them.

### After Finishing an Assignment
1. Delete your code and rewrite it from memory. You'll be surprised what you don't actually know.
2. Write 3 bullet points: what you learned, what confused you, and one question you still have.
3. Look up the official documentation for the main library used (Node.js docs, Express.js docs, etc.).

---

## How to Approach Projects

Projects are integrations — they connect concepts from multiple assignments. They are harder than assignments by design.

### Step 1 — Plan Before You Code (15 minutes)
- Draw the file structure on paper or a whiteboard.
- Write down the API endpoints and their expected request/response shapes.
- Identify which assignment concepts apply to which parts.

### Step 2 — Build the Skeleton First
Create empty files with placeholder exports:
```js
// routes/recipes.js
const router = require('express').Router();
// TODO: implement routes
module.exports = router;
```
Get the app to start without errors before writing logic.

### Step 3 — Implement Feature by Feature
Complete one endpoint at a time. Test it with curl or Postman/Insomnia before moving on.

### Step 4 — Review Deliverables
Go through the deliverables checklist line by line. If something is unchecked, implement it.

---

## Tools You Need

### Required
- **Node.js** 20+ (use `nvm` to manage versions: `nvm install 20 && nvm use 20`)
- **npm** or **pnpm** (pnpm is faster: `npm install -g pnpm`)
- **VS Code** with extensions: ESLint, Prettier, REST Client or Thunder Client
- **Git** — commit after every assignment
- **curl** or **Postman/Insomnia** for testing APIs

### Recommended
- **Docker Desktop** — needed from Phase 2 onwards for running databases locally
- **TablePlus** or **DBeaver** — GUI for PostgreSQL
- **MongoDB Compass** — GUI for MongoDB
- **Redis Insight** — GUI for Redis

---

## Running Databases Locally

Use Docker to avoid polluting your machine with multiple database installs:

```bash
# MongoDB
docker run -d --name mongo -p 27017:27017 mongo:7

# PostgreSQL
docker run -d --name postgres -p 5432:5432 \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=devdb \
  postgres:16

# Redis
docker run -d --name redis -p 6379:6379 redis:7-alpine
```

Or use a single `docker-compose.yml` in your workspace root.

---

## Project Structure Convention

Use this structure for every project you build:

```
project-name/
├── src/
│   ├── app.js          # Express app (no listen())
│   ├── server.js       # Entry point (calls app.listen())
│   ├── config/         # Environment, database config
│   ├── routes/         # Express routers
│   ├── controllers/    # Route handler functions
│   ├── services/       # Business logic
│   ├── models/         # Database models/schemas
│   ├── middleware/      # Custom middleware
│   └── utils/          # Helper functions
├── tests/
├── .env
├── .env.example
├── .gitignore
└── package.json
```

Keeping `app.js` separate from `server.js` makes testing much easier — you can import `app` in tests without starting the HTTP server.

---

## JavaScript Style Rules

Follow these rules throughout the curriculum. Consistency matters more than the specific choices.

```js
// Use ES modules (type: "module" in package.json) OR CommonJS consistently
// This curriculum uses CommonJS (require/module.exports) for simplicity

// Always use async/await — never raw callbacks
// Bad:
fs.readFile('file.txt', (err, data) => { ... });
// Good:
const data = await fs.promises.readFile('file.txt', 'utf8');

// Use const by default, let when reassignment is needed, never var

// Destructure where it improves readability
const { name, email } = req.body;

// Always handle async errors with try/catch or asyncWrapper
```

---

## When You Get Stuck

Use this escalation order. Only move to the next step if the previous didn't resolve it after 10 minutes:

1. **Re-read the error message** — most errors tell you exactly what's wrong.
2. **Console.log the problem area** — inspect the actual values, not the expected ones.
3. **Check the official docs** — not Stack Overflow first.
4. **Search the specific error message** on Google (include the library name and version).
5. **Take a 10-minute break** — this solves problems surprisingly often.
6. **Rubber duck debug** — explain the problem out loud as if to someone unfamiliar with it.

---

## Git Discipline

Commit after every completed assignment. Use descriptive messages:

```bash
git init
git add .
git commit -m "A1.1: Node.js core — streams and EventEmitter"
git commit -m "A1.2: Express basics — routing and middleware"
git commit -m "P1.1: Recipe API complete"
```

This gives you a history to refer back to and makes it easy to see your progress.

---

## What "Done" Means

An assignment is done when:
- [ ] All code in the assignment runs without errors
- [ ] You can explain every line you wrote
- [ ] You rewrote it once from memory
- [ ] You've read the relevant official docs section

A project is done when:
- [ ] All deliverables are checked
- [ ] The app runs end-to-end with no errors
- [ ] You've tested every endpoint manually
- [ ] Code is committed to git with a descriptive message

---

## Progression Advice

### If you're a beginner to Node.js
Spend extra time on Phase 1. Re-read the Node.js event loop explanations multiple times. Everything else builds on understanding why Node.js is single-threaded but non-blocking.

### If you already know Express basics
You can skim Phase 1 assignments but still build the projects. The projects in Phase 1 enforce good architecture habits you'll use throughout.

### If you're preparing for interviews
Phase 3 (auth), Phase 4 (error handling), and Phase 7 (testing) are the most commonly tested in backend interviews. Make sure you can build a JWT auth flow and write test suites from scratch.

### If you're preparing for production work
Phases 5, 8, and 9 are the most important for real-world backend roles. Employers care that you know Docker, CI/CD, and can monitor a running service.

---

## Tracking Progress

Keep a `progress.md` in your workspace:

```markdown
## Progress Log

### Week 1
- [x] A1.1 — Node.js core (2h)
- [x] A1.2 — Express basics (1.5h)
- [ ] A1.3 — Middleware
- [ ] A1.4 — Routing organization
- [ ] P1.1 — Recipe API
- [ ] P1.2 — Notes API
```

Estimate time taken for each item. Over time you'll get better at knowing how long things take, which is a professional skill in itself.
