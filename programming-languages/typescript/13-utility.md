# 13 - Utility Types

Built-in generic types that transform other types. High-frequency in interviews and real code.

Assume this base type for examples:

```ts
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;
}
```

## Object shape transforms

### `Partial<T>` — all optional

```ts
type UserUpdate = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number }
```

Use for update payloads / patch operations.

### `Required<T>` — all required

```ts
type FullUser = Required<User>;
// age is now required
```

### `Readonly<T>` — all readonly

```ts
type FrozenUser = Readonly<User>;
// frozen.id = 2;   // Error
```

### `Pick<T, K>` — keep only listed keys

```ts
type UserPublic = Pick<User, "id" | "name">;
```

### `Omit<T, K>` — remove listed keys

```ts
type UserNoEmail = Omit<User, "email">;
```

### `Record<K, V>` — key→value map

```ts
type Roles = Record<"admin" | "user" | "guest", boolean>;
// { admin: boolean; user: boolean; guest: boolean }
```

Prefer `Record` over index signatures when keys are known.

## Union / nullability transforms

### `Exclude<T, U>` — remove union members assignable to `U`

```ts
type T = "a" | "b" | "c";
type X = Exclude<T, "a">;          // "b" | "c"
type Y = Exclude<string | number | null, null>;  // string | number
```

### `Extract<T, U>` — keep only assignable members

```ts
type A = Extract<string | number | boolean, boolean | string>;
// string | boolean
```

### `NonNullable<T>` — remove `null` and `undefined`

```ts
type S = string | null | undefined;
type N = NonNullable<S>;   // string
```

## Function-related

### `Parameters<T>` — tuple of a function's parameter types

```ts
function fn(a: number, b: string) {}
type P = Parameters<typeof fn>;   // [number, string]
```

### `ReturnType<T>` — the return type

```ts
type R = ReturnType<typeof fn>;   // void
```

Pairs well with `typeof` for API hooks, higher-order functions.

### `ConstructorParameters<T>`

```ts
class Thing { constructor(public x: number, public y: string) {} }
type C = ConstructorParameters<typeof Thing>;   // [number, string]
```

### `InstanceType<T>`

```ts
type I = InstanceType<typeof Thing>;   // Thing
```

### `ThisParameterType<T>` / `OmitThisParameter<T>`

Less common; used for typing libraries that manipulate `this`.

## String literal transforms

### `Uppercase<T>`, `Lowercase<T>`, `Capitalize<T>`, `Uncapitalize<T>`

```ts
type E = Uppercase<"hello">;      // "HELLO"
type C = Capitalize<"hello">;     // "Hello"
```

Combines nicely with template literal types:

```ts
type Event = "click" | "hover";
type Handler = `on${Capitalize<Event>}`;   // "onClick" | "onHover"
```

## Promise helper

### `Awaited<T>` — unwrap Promises

```ts
type A = Awaited<Promise<Promise<string>>>;   // string
```

Useful for typed `await` results of generic promises.

## Patterns / combinations

### Make some keys optional

```ts
type PartialBy<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type U = PartialBy<User, "email" | "name">;
```

### Make some keys required

```ts
type RequiredBy<T, K extends keyof T> = T & Required<Pick<T, K>>;
```

### Deep readonly (manual, since there's no built-in)

```ts
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};
```

### Value types of an object

```ts
type Values<T> = T[keyof T];
type V = Values<{ a: 1; b: "x" }>;   // 1 | "x"
```

## Quick reference table

| Utility | Does what |
|---|---|
| `Partial<T>` | All keys optional |
| `Required<T>` | All keys required |
| `Readonly<T>` | All keys readonly |
| `Pick<T, K>` | Keep keys `K` |
| `Omit<T, K>` | Drop keys `K` |
| `Record<K, V>` | Object with keys `K`, values `V` |
| `Exclude<T, U>` | Remove `U`-assignable from `T` |
| `Extract<T, U>` | Keep `U`-assignable from `T` |
| `NonNullable<T>` | Remove `null` \| `undefined` |
| `Parameters<T>` | Tuple of params |
| `ReturnType<T>` | Return type |
| `ConstructorParameters<T>` | Ctor params tuple |
| `InstanceType<T>` | Instance of ctor |
| `Awaited<T>` | Unwrap Promises |
| `Uppercase/Lowercase/Capitalize/Uncapitalize<T>` | String transforms |

## Interview Q&A

1. **Difference between `Pick` and `Omit`?** — `Pick` keeps listed keys; `Omit` removes them.
2. **`Partial` vs `Required`?** — Opposites — makes keys optional vs required.
3. **How would you type a PATCH request body?** — `Partial<T>` (optionally intersected with required id).
4. **How to derive a function's return type?** — `ReturnType<typeof fn>`.
5. **What is `Awaited<T>` for?** — Recursively unwraps Promise types, mirroring `await` in types.
