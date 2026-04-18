# 16 - DefinitelyTyped, `@types`, and Declaration Files

Most JS libraries are untyped. **DefinitelyTyped** and `.d.ts` files bridge that gap so TS users can consume plain JS with full type safety.

## What is DefinitelyTyped?

- A **community-maintained GitHub repo** ([DefinitelyTyped/DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)).
- Contains **TypeScript declaration files** (`.d.ts`) for thousands of JS libraries (lodash, express, jquery, node, react — before they shipped types, etc.).
- Declarations are published to npm under the scope `@types/<package>`.

```bash
npm install --save-dev @types/lodash
npm install --save-dev @types/node
npm install --save-dev @types/express
```

After installing, `import _ from "lodash"` is fully typed — no changes needed in the library itself.

## Declaration files — `.d.ts`

Ambient **type-only** files — no runtime code, only type information.

```ts
// math.d.ts
declare module "math-lib" {
  export function add(a: number, b: number): number;
  export const PI: number;
}
```

Generated automatically when you compile a TS library with `"declaration": true` in `tsconfig.json`.

## How TS finds types

When you `import "foo"`, the compiler looks (in order):

1. `foo/package.json` → `"types"` / `"typings"` field.
2. `foo/index.d.ts`.
3. `node_modules/@types/foo` (a `@types` package).
4. Any file that declares `module "foo"` in the project.
5. Falls back to `any` (with `noImplicitAny: false`) or errors.

### `typeRoots` and `types`

```json
{
  "compilerOptions": {
    "typeRoots": ["./node_modules/@types", "./types"],
    "types": ["node", "jest"]
  }
}
```

- `typeRoots` — folders to scan for type packages.
- `types` — restrict which `@types/*` packages are globally included (optimization).

## Writing your own declarations

### Ambient declaration for a missing module

If you must use a library with no types:

```ts
// types/untyped-lib.d.ts
declare module "untyped-lib";          // treats whole module as any
```

Better — supply real types:

```ts
declare module "untyped-lib" {
  export function doThing(x: number): string;
  const version: string;
  export default version;
}
```

Make sure the file is in `include` or `typeRoots`.

### Global types / globals

```ts
// types/global.d.ts
declare global {
  interface Window {
    myApp: { version: string };
  }
}
export {};   // makes it a module
```

Now `window.myApp.version` is typed everywhere.

### Non-JS asset modules (images, SVGs, CSS modules)

Very common in React/webpack projects:

```ts
// types/assets.d.ts
declare module "*.svg" {
  const content: string;
  export default content;
}

declare module "*.module.css" {
  const classes: Record<string, string>;
  export default classes;
}
```

### Augmenting third-party modules

```ts
import "express";

declare module "express-serve-static-core" {
  interface Request {
    user?: { id: string };
  }
}
```

This relies on TS's **declaration merging** — one of the main reasons to prefer `interface` for library types.

## Publishing a typed library

In your library's `tsconfig.json`:

```json
{
  "compilerOptions": {
    "declaration": true,
    "declarationMap": true,
    "outDir": "dist"
  }
}
```

In `package.json`:

```json
{
  "main": "dist/index.js",
  "types": "dist/index.d.ts"
}
```

Consumers now get types automatically — no `@types` package needed. Modern libraries ship types this way.

## Common commands

```bash
# Install types
npm i -D @types/node @types/react

# Check what @types packages exist
npm search @types/<name>

# Update types
npm outdated @types/<name>
```

## DefinitelyTyped contribution tips

- Each package lives in `DefinitelyTyped/types/<package>/index.d.ts`.
- Must pass `dtslint` (a type-level lint) + tests.
- Versioned to match the JS package's major version.
- Merged via PR to the DT repo; publishes automatically to npm as `@types/<pkg>`.

## Interview Q&A

1. **What is DefinitelyTyped?** — Community repo of `.d.ts` files for JS libraries, published as `@types/*` on npm.
2. **How do I add types for a JS library?** — Check for `@types/<lib>`; if none exists, write a `.d.ts` file declaring the module.
3. **What is a `.d.ts` file?** — A declaration-only file with no emitted JS — contains types, signatures, ambient declarations.
4. **How does TS locate types for an import?** — Package's `types` field → `index.d.ts` → `@types/<pkg>` → ambient declarations in the project.
5. **What does `declare module "*.svg"` do?** — Declares a wildcard module so imports of `.svg` files are typed.
6. **How do I augment an existing library's types?** — Use `declare module` + interface merging (e.g. add a property to `Express.Request`).
