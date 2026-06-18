# How to Learn From This Curriculum

---

## The Learning Loop

Every phase follows the same loop:

```
Read assignment → Run the code → Break it → Fix it → Do the project
```

Do not skip the "break it" step. The moment you understand *why* something fails, you understand the concept at a deeper level than any tutorial can teach.

---

## How to Approach Assignments

Each assignment (`A{phase}.{num}.md`) teaches a specific NestJS concept.

1. **Read the Background section first** — do not skip theory. NestJS has opinions; understanding why it was designed a certain way prevents confusion later.
2. **Type the code by hand** — do not copy-paste. Muscle memory matters for decorators and module wiring.
3. **Run each step before moving to the next** — NestJS gives excellent error messages. Read them fully.
4. **Check the Deliverables list** — tick each item only when you can explain *why* it works, not just that it works.

### Time budget per assignment
- Read background: 15–20 min
- Implement tasks: 45–90 min
- Verify deliverables: 15–20 min

---

## How to Approach Projects

Each project (`P{phase}.{num}.md`) integrates everything from the current and prior phases.

1. **Set up the directory tree first** — create every folder and empty file before writing a single line of logic. This forces you to think about architecture upfront.
2. **Write module files before service files** — NestJS bootstraps via modules; wiring comes first.
3. **Test each layer as you build it** — don't build all three layers then debug all three.
4. **Use the Architecture diagram** — if your actual structure diverges, update the diagram. Keeping it accurate trains the habit of documentation.

### Time budget per project
- Setup + architecture: 30 min
- Core implementation: 2–4 hours
- Edge cases + validation: 1–2 hours
- Deliverables review: 30 min

---

## Environment Setup

### 1. Install tools

```bash
node --version   # must be >= 18
npm i -g @nestjs/cli
nest --version   # must be >= 10
```

### 2. Create a new project (each assignment gets its own)

```bash
nest new phase1-fundamentals
cd phase1-fundamentals
npm run start:dev
```

### 3. Docker for databases

Create a `docker-compose.yml` at the root of every project:

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: nestuser
      POSTGRES_PASSWORD: nestpass
      POSTGRES_DB: nestdb
    ports:
      - '5432:5432'
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - '6379:6379'

volumes:
  pgdata:
```

```bash
docker-compose up -d
```

### 4. Recommended VSCode extensions

- **ESLint** — catches decorator misuse early
- **Prettier** — consistent formatting
- **NestJS Files** — scaffold modules/controllers/services via right-click
- **Thunder Client** or **REST Client** — test endpoints without leaving VSCode

---

## NestJS Mental Models

### The Module Graph

NestJS builds a directed acyclic graph of modules at startup. Every provider lives in exactly one module. To use a provider in another module, the owning module must **export** it and the consuming module must **import** the owning module.

```
AppModule
  └── UsersModule (exports: UsersService)
  └── AuthModule  (imports: UsersModule) → can inject UsersService
  └── PostsModule (imports: UsersModule) → can inject UsersService
```

### Dependency Injection Flow

```
NestJS Runtime
  → reads @Module metadata
  → builds provider tokens
  → resolves constructor dependencies recursively
  → instantiates providers (singletons by default)
  → injects into controllers
```

### Request Lifecycle

```
Request
  → Middleware
  → Guards
  → Interceptors (before)
  → Pipes
  → Controller Handler
  → Interceptors (after)
  → Exception Filters (only on error)
  → Response
```

Memorise this order. Every phase builds on it.

---

## Debugging Tips

### Provider not found
```
Nest can't resolve dependencies of the XxxService (?).
```
- Check that `XxxModule` is imported in the consuming module.
- Check that `XxxService` is in the `providers` array of `XxxModule`.
- Check that `XxxModule` exports `XxxService` if used outside.

### Circular dependency
```
A circular dependency between modules detected.
```
- Use `forwardRef(() => XxxModule)` in the imports array of both modules.
- Better: refactor to extract shared logic into a third module.

### Validation not running
- Ensure `ValidationPipe` is applied globally via `app.useGlobalPipes(new ValidationPipe())`.
- Ensure `@Body()`, `@Query()`, or `@Param()` decorators are present.
- Ensure the DTO class uses `class-validator` decorators (not interfaces).

---

## Code Style Conventions (used throughout this curriculum)

```typescript
// Module files: xxx.module.ts
// Controller files: xxx.controller.ts
// Service files: xxx.service.ts
// DTO files: dto/create-xxx.dto.ts, dto/update-xxx.dto.ts
// Entity files: entities/xxx.entity.ts (TypeORM) or included in schema.prisma
// Guard files: guards/xxx.guard.ts
// Interceptor files: interceptors/xxx.interceptor.ts
// Filter files: filters/xxx.filter.ts
// Decorator files: decorators/xxx.decorator.ts
```

All imports use absolute paths configured in `tsconfig.json`:
```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

---

## Progress Tracking

Use this checklist. Mark a phase complete only when all projects pass their deliverables.

- [ ] Phase 1 — NestJS Fundamentals
- [ ] Phase 2 — Database with TypeORM & Prisma
- [ ] Phase 3 — Authentication & Guards
- [ ] Phase 4 — Pipes, Interceptors & Filters
- [ ] Phase 5 — Configuration & Environment
- [ ] Phase 6 — Caching, Queues & Performance
- [ ] Phase 7 — Testing
- [ ] Phase 8 — Microservices
- [ ] Phase 9 — Advanced & Production

---

## When You Get Stuck

1. Read the NestJS official docs first: https://docs.nestjs.com
2. Search the NestJS GitHub issues — most configuration problems are already answered.
3. Check the NestJS Discord: https://discord.gg/nestjs
4. Ask with a minimal reproduction — isolate the problem to the smallest possible module.

Do not spend more than 30 minutes stuck on the same issue without reaching out. Unblocking yourself quickly is a professional skill.
