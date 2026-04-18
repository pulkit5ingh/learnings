# 15 - `null`, `undefined` & Strictness

Handling "absent" values is one of the biggest wins of TypeScript over JavaScript. The compiler flag `strictNullChecks` decides how strict this is.

## `null` vs `undefined`

| | `undefined` | `null` |
|---|---|---|
| Meaning | "not assigned yet" (JS default) | "intentionally empty" |
| Type | `undefined` | `null` |
| `typeof` | `"undefined"` | `"object"` (historical quirk) |
| Default function param | yes | no |
| JSON | dropped / unsupported | supported |

**Convention:** use `undefined` within TS; reserve `null` for JSON APIs that use it explicitly.

## `strictNullChecks`

Part of `"strict": true`. When enabled:

- `null` and `undefined` are *not* assignable to other types.
- They must be **explicit** in the type (usually via a union).

```ts
// strictNullChecks: true
let s: string = null;          // Error
let s2: string | null = null;  // ok
```

When disabled (not recommended), `null`/`undefined` silently sneak into every type — the "billion-dollar mistake" returns.

## Optional properties & parameters

Optional marker `?` adds `undefined` automatically.

```ts
interface User {
  id: number;
  nick?: string;       // string | undefined
}

function f(x?: number) { /* x: number | undefined */ }
```

**Note:** `x?: number` means "may be omitted" — you cannot pass `null` explicitly unless the type is `number | null | undefined`.

## Narrowing null / undefined

```ts
function len(s: string | null): number {
  if (s === null) return 0;
  return s.length;                 // s: string after narrow
}

function len2(s?: string): number {
  if (!s) return 0;                // narrows out undefined and ""
  return s.length;
}
```

Truthy narrow (`if (s)`) also removes empty string, `0`, `NaN` — not always what you want. Prefer explicit `!= null`.

### `x != null` trick

```ts
if (x != null) {
  // x is neither null nor undefined (loose equality covers both)
}
```

Considered clean and idiomatic for removing both nullish values.

## Optional chaining — `?.`

Short-circuits to `undefined` if the target is `null`/`undefined`.

```ts
const city = user?.address?.city;  // string | undefined
user.list?.();                     // call only if exists
arr?.[0];                          // index only if exists
```

## Nullish coalescing — `??`

Returns right-hand side only if left is `null`/`undefined` (not `0`, `""`, `false`).

```ts
const port = cfg.port ?? 3000;
const title = input ?? "Untitled";

// vs `||` which also replaces "", 0, false
const wrong = count || 10;     // replaces 0 too
```

## Non-null assertion — `!`

Tells the compiler "this is definitely not null/undefined" — no runtime check.

```ts
const el = document.getElementById("root")!;   // HTMLElement (not null)
```

Use sparingly; prefer narrowing.

## Definite assignment — `!:` on fields

For class fields assigned outside the constructor.

```ts
class Service {
  client!: HttpClient;   // set by DI later
}
```

## Return types involving null

```ts
function find(id: number): User | undefined { /* ... */ }
function findOrThrow(id: number): User {
  const u = find(id);
  if (!u) throw new Error("not found");
  return u;
}
```

## `void` vs `undefined`

- `void` — function return "you shouldn't use this".
- `undefined` — the actual value `undefined`.

```ts
function a(): void {}            // callers shouldn't inspect the return
function b(): undefined { return undefined; }   // explicit
```

A function typed `(): void` can still return a value; it's just ignored. This makes `Array.prototype.forEach` callbacks convenient.

## Arrays & objects with null

```ts
const arr: (string | null)[] = ["a", null, "b"];
const filtered = arr.filter((s): s is string => s !== null);   // string[]
```

The type predicate `s is string` makes `filter` narrow properly.

## `Maybe<T>` / `Nullable<T>` alias

```ts
type Maybe<T> = T | null | undefined;
type Nullable<T> = T | null;

function getUser(id: number): Maybe<User> { /* ... */ }
```

## Patterns

### Default via `??`

```ts
function greet(name?: string) {
  return `Hi, ${name ?? "stranger"}`;
}
```

### Guard clauses over nesting

```ts
function load(id?: string) {
  if (!id) return;
  fetchUser(id);   // id: string here
}
```

### Exhaustiveness with discriminated unions

See `08-union.md`.

## Interview Q&A

1. **`null` vs `undefined`?** — `undefined` is the JS default for "not set"; `null` is an explicit "no value".
2. **What does `strictNullChecks` do?** — Disallows `null`/`undefined` in types unless explicitly unioned in.
3. **`??` vs `||`?** — `??` only replaces `null`/`undefined`; `||` replaces any falsy (`0`, `""`, `false`).
4. **Optional chaining short-circuit?** — `a?.b` returns `undefined` if `a` is nullish, otherwise `a.b`.
5. **When is `!` (non-null assertion) acceptable?** — When you have out-of-band knowledge the compiler can't see (e.g. DOM element you just created) and narrowing is impractical.
6. **`void` vs `undefined` return type?** — `void` = ignore return; `undefined` = must be the value `undefined`.
