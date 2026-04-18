# 05 - Object Types in TypeScript

Objects are unordered key-value collections. In TS, the **shape** (keys + their types) becomes the type.

## Inline object types

```ts
let user: { id: number; name: string } = { id: 1, name: "Pulkit" };
```

## Optional properties — `?`

```ts
type User = { id: number; name: string; email?: string };
const u: User = { id: 1, name: "Pulkit" };   // email omitted — OK
```

## Readonly properties

```ts
type Config = { readonly apiUrl: string };
const c: Config = { apiUrl: "/api" };
// c.apiUrl = "/v2";   // Error
```

Shallow only — nested objects are still mutable unless also marked readonly.

## Index signatures — dynamic keys

When keys aren't known ahead of time.

```ts
type ScoreMap = { [player: string]: number };
const scores: ScoreMap = { alice: 90, bob: 88 };

// Mixed: known + dynamic
type Dict = {
  length: number;
  [key: string]: string | number;
};
```

- Index signatures enforce that **every** key must match the value type (including the known ones).

## `Record<K, V>` — preferred helper

```ts
type RoleMap = Record<"admin" | "user", boolean>;
const m: RoleMap = { admin: true, user: false };
```

More ergonomic than index signatures when keys are known.

## Nested objects

```ts
type Address = { city: string; zip: string };
type User = {
  id: number;
  address: Address;
};
```

## Methods on objects

```ts
type Greeter = {
  name: string;
  greet(): string;                 // shorthand
  shout: (msg: string) => string;  // property form
};
```

## Object destructuring with types

```ts
function printUser({ id, name }: { id: number; name: string }) {
  console.log(id, name);
}

// Or use a named type
type User = { id: number; name: string };
function print({ id, name }: User) { /* ... */ }
```

## Excess property checks

Object literals assigned directly are checked for extra keys.

```ts
type U = { id: number };
// const u: U = { id: 1, extra: "x" };   // Error — extra property

const raw = { id: 1, extra: "x" };
const u: U = raw;                        // ok (no literal)
```

## `object` vs `{}` vs `Record<string, unknown>`

| Type | Meaning |
|---|---|
| `object` | Any non-primitive (array, function, object). Rarely precise enough. |
| `{}` | Any value *except* `null`/`undefined` — very loose. |
| `Record<string, unknown>` | A plain keyed object (recommended). |
| `Record<string, any>` | Same but disables type safety on values. |

Prefer a **specific shape** or `Record<string, unknown>`.

## Structural typing (duck typing)

TS uses **structural** typing — if the shape matches, it's assignable.

```ts
type Point = { x: number; y: number };
const p = { x: 1, y: 2, z: 3 };
const q: Point = p;   // ok — has x and y
```

Contrast with nominal typing (Java/C#) where names matter.

## Extending / merging object types

### Using `interface`

```ts
interface Base { id: number }
interface User extends Base { name: string }
```

### Using `type` + intersection

```ts
type Base = { id: number };
type User = Base & { name: string };
```

## Object utility types (preview)

- `Partial<T>` — all props optional.
- `Required<T>` — all props required.
- `Readonly<T>` — all props readonly.
- `Pick<T, K>` — subset of keys.
- `Omit<T, K>` — all keys except.

(Covered in `13-utility.md`.)

## Interview Q&A

1. **What is structural typing?** — Compatibility based on shape, not name.
2. **`type` vs `interface` for object shapes?** — Both work; interfaces can be declaration-merged/extended, types can use unions/intersections/mapped types.
3. **Why excess property checks?** — TS blocks unknown keys on literals to catch typos.
4. **Index signature vs `Record`?** — Equivalent, but `Record<K, V>` is cleaner when `K` is known.
5. **Does `readonly` deep-freeze?** — No, it's shallow. Use `Readonly<T>` or deep-readonly types for nested.
