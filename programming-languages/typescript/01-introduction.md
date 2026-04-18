# 01 - Introduction to TypeScript

> Reference: [W3Schools TypeScript](https://www.w3schools.com/typescript/index.php) · [Official Docs](https://www.typescriptlang.org/docs/)

## What is TypeScript?

TypeScript (TS) is a **strongly-typed, object-oriented, compiled superset of JavaScript** developed by Microsoft (2012). It adds **static typing**, **interfaces**, **generics**, **enums**, and modern JS features on top of plain JavaScript.

- TS code is **transpiled** (compiled) into plain JS by the TypeScript compiler (`tsc`).
- Browsers / Node.js do **not** understand TS directly — they run the emitted JS.
- Every valid JS file is a valid TS file (hence "superset").

## TypeScript vs JavaScript

| Feature | JavaScript | TypeScript |
|---|---|---|
| Typing | Dynamic (types checked at runtime) | Static (types checked at compile time) |
| Errors | Found at runtime | Found at compile time |
| Compilation | Interpreted directly | Must be compiled to JS |
| Interfaces / Generics | No | Yes |
| Enums, Tuples, Decorators | No | Yes |
| Tooling / IntelliSense | Limited | Rich (autocomplete, refactoring, navigation) |
| Learning curve | Easier | Slightly steeper |
| Runs in browser | Yes | No (needs compilation) |

**Interview take:** *"TypeScript is a static-typed superset of JavaScript that compiles to JS. Its main benefit is catching type-related bugs at compile time and providing better IDE tooling, which improves maintainability in large codebases."*

## Why use TypeScript?

- **Early bug detection** — catches `undefined is not a function` at compile time.
- **Self-documenting code** — types act as inline documentation.
- **Refactoring confidence** — rename/move safely across large projects.
- **Better IDE support** — autocomplete, go-to-definition, inline errors.
- **Scales** — preferred for medium/large apps (Angular, NestJS, VS Code, Slack, Airbnb).

## TS and JS Interoperability

- You can import `.js` files from `.ts` (with `allowJs: true`).
- You can gradually migrate a JS codebase file-by-file.
- JS libraries without types can still be used via:
  - `declare module "lib"` — quick ambient declaration
  - `@types/<lib>` packages from DefinitelyTyped
  - `any` as an escape hatch (avoid in production)
- TS emits plain JS that any JS runtime can execute.

## Installation

```bash
# global
npm install -g typescript

# project-local (recommended)
npm install --save-dev typescript

# check version
tsc --version
```

## Running TypeScript

```bash
tsc file.ts              # compiles file.ts -> file.js
tsc                      # compiles whole project using tsconfig.json
tsc --watch              # recompile on file changes
npx ts-node file.ts      # run TS directly (dev only)
```

- **`tsc`** — official compiler; produces `.js` output.
- **`ts-node`** — runs TS in Node without an explicit build step (great for scripts).
- **TS Playground** — [typescriptlang.org/play](https://www.typescriptlang.org/play/) for quick experiments.

## Configuration — `tsconfig.json`

Generated via `tsc --init`. Tells the compiler *how* to compile your project.

```json
{
  "compilerOptions": {
    "target": "ES2020",          // JS version to emit
    "module": "commonjs",        // module system (commonjs | esnext | nodenext)
    "strict": true,              // enables ALL strict checks (recommended)
    "esModuleInterop": true,     // allow `import x from "cjs-lib"`
    "skipLibCheck": true,        // skip type-check of .d.ts files (faster)
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "sourceMap": true,           // generate .map for debugging
    "declaration": true,         // emit .d.ts files (for libs)
    "resolveJsonModule": true,
    "moduleResolution": "node"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Important compiler options (interview favourites)

| Option | Purpose |
|---|---|
| `strict` | Umbrella for `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, etc. |
| `noImplicitAny` | Error when TS can't infer a type and would fall back to `any`. |
| `strictNullChecks` | `null` and `undefined` are *not* assignable to other types. |
| `target` | ECMAScript version of emitted JS. |
| `module` | Module system of emitted JS. |
| `lib` | Which built-in type libs to include (e.g. `DOM`, `ES2020`). |
| `baseUrl` / `paths` | Path aliases like `@/components/*`. |
| `noUnusedLocals` / `noUnusedParameters` | Lint-like checks. |
| `noEmitOnError` | Don't emit JS if compile errors exist. |

## Common interview questions

1. **What is TypeScript? Why use it over JavaScript?**
2. **Is TypeScript a replacement for JavaScript?** — No, it compiles *to* JavaScript.
3. **What is `tsconfig.json`?** — Project-level compiler configuration.
4. **What does `strict: true` do?** — Enables all strict type-checking options.
5. **Difference between `tsc` and `ts-node`?** — `tsc` compiles to `.js`; `ts-node` runs TS in-memory.
6. **Can TS run in the browser directly?** — No, must be compiled first.
