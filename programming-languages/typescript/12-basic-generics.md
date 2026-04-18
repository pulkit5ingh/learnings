# 12 - Generics

Generics let you write **reusable, type-safe** code that works across many types, without losing type information via `any`.

## Motivation

```ts
// Without generics — loses type info
function identity(x: any): any { return x; }
const a = identity("hi");          // a: any (bad)

// With generics
function identity<T>(x: T): T { return x; }
const b = identity("hi");          // b: string
const c = identity<number>(42);    // explicit
```

`T` is a **type parameter** — a placeholder for the actual type provided at call site.

## Generic functions

```ts
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}

first([1, 2, 3]);          // number | undefined
first(["a", "b"]);         // string | undefined
```

TS infers `T` from arguments. You can specify explicitly: `first<string>([...])`.

### Multiple type parameters

```ts
function pair<K, V>(k: K, v: V): [K, V] { return [k, v]; }

const p = pair("id", 1);   // [string, number]
```

## Generic constraints — `extends`

Restrict what `T` can be.

```ts
function length<T extends { length: number }>(x: T): number {
  return x.length;
}

length("hi");        // ok
length([1, 2]);      // ok
// length(10);        // Error — number has no length
```

### `keyof` constraints

```ts
function pick<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const u = { id: 1, name: "a" };
pick(u, "id");     // number
// pick(u, "xx");   // Error — not a key
```

## Default type parameters

```ts
function createList<T = string>(): T[] { return []; }

const a = createList();          // string[]
const b = createList<number>();  // number[]
```

## Generic interfaces

```ts
interface ApiResponse<T> {
  ok: boolean;
  data: T;
  error?: string;
}

const r: ApiResponse<User> = { ok: true, data: user };
```

## Generic type aliases

```ts
type Dict<V> = Record<string, V>;
type Maybe<T> = T | null | undefined;
```

## Generic classes

```ts
class Stack<T> {
  private items: T[] = [];
  push(x: T) { this.items.push(x); }
  pop(): T | undefined { return this.items.pop(); }
}

const s = new Stack<number>();
s.push(1);
```

## Generic methods on a non-generic class

```ts
class Registry {
  get<T>(key: string): T { return {} as T; }
}
```

## Generic constraints + defaults combined

```ts
interface Paged<T = unknown, P extends number = 10> {
  items: T[];
  pageSize: P;
}
```

## Higher-kinded use cases (real-world)

### Typed HTTP client

```ts
async function fetchJson<T>(url: string): Promise<T> {
  const res = await fetch(url);
  return (await res.json()) as T;
}

const user = await fetchJson<User>("/api/users/1");
```

### React-style hook

```ts
function useState<T>(initial: T): [T, (v: T) => void] {
  let value = initial;
  return [value, (v: T) => { value = v; }];
}
```

## Variance & inference tips

- Let TS **infer** generics from arguments — specify only when needed.
- A generic with no relationship to its inputs/outputs is suspect (probably should be a concrete type).
- Use `extends` constraints to access members on `T` inside the function.

## Generics vs `any`

- `any` — erases types, anything goes.
- Generics — preserve the caller's type through to the return.

```ts
function idAny(x: any): any { return x; }
function idGen<T>(x: T): T { return x; }

idAny("hi").toUpperCase();    // works, but no type info
idGen("hi").toUpperCase();    // works AND keeps type
```

## Interview Q&A

1. **What are generics and why use them?** — Type parameters that preserve type information across reusable code — safer than `any`.
2. **Difference between `T extends X` and `T: X`?** — TS uses `extends` for constraints; there's no `:` in generic parameters.
3. **What is a default type parameter?** — `<T = string>`; used if caller doesn't provide the type.
4. **Why constrain `<T extends object>`?** — Lets you access object-level operations on `T` inside the function.
5. **Can generics appear on classes / interfaces / aliases?** — Yes, all three.
