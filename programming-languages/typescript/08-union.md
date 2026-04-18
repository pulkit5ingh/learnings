# 08 - Union, Intersection & Literal Types

Three of the most powerful features of TS's type system.

## 1. Union types — `A | B`

A value can be one of several types.

```ts
let id: string | number;
id = "abc";
id = 1;
// id = true;   // Error
```

You can only access members that are valid for **every** member of the union — the rest require **narrowing**.

```ts
function pad(v: string | number): string {
  // v.toFixed();   // Error — not on string
  if (typeof v === "number") return v.toFixed(2);
  return v.padStart(5, "0");
}
```

### Narrowing techniques

- `typeof v === "string"` — for primitives
- `v instanceof Date` — for classes
- `"prop" in v` — for object shapes
- Equality checks: `v === null`
- User-defined **type guards**: `function isUser(x): x is User { ... }`

### Discriminated (tagged) unions — interview favourite

Attach a literal "tag" field that TS uses to narrow.

```ts
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; side: number }
  | { kind: "rect"; w: number; h: number };

function area(s: Shape): number {
  switch (s.kind) {
    case "circle": return Math.PI * s.radius ** 2;
    case "square": return s.side ** 2;
    case "rect":   return s.w * s.h;
    default: {
      const _: never = s;   // exhaustiveness
      return _;
    }
  }
}
```

## 2. Intersection types — `A & B`

Combine multiple types into one with **all** members.

```ts
type Name = { name: string };
type Age  = { age: number };
type Person = Name & Age;

const p: Person = { name: "A", age: 1 };
```

### Conflicting members → `never`

```ts
type Bad = { x: string } & { x: number };   // x: never
```

### Mixins and composition

```ts
type WithTimestamps = { createdAt: Date; updatedAt: Date };
type User = { id: number; name: string } & WithTimestamps;
```

## 3. Literal types

A type whose only valid value is a specific literal.

```ts
let dir: "N" | "S" | "E" | "W";
dir = "N";
// dir = "X";   // Error

let zero: 0 = 0;
let flag: true = true;
```

Combined with unions, they replace many enum use cases.

```ts
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
```

### Literal inference and `as const`

```ts
const x = "hello";             // inferred: "hello"
let   y = "hello";             // inferred: string (widened)

const cfg = { dir: "N" };      // cfg.dir: string
const cfgC = { dir: "N" } as const;  // cfg.dir: "N"
```

`as const` is crucial to keep literal narrowness.

## 4. Template literal types

Compose string literal types.

```ts
type Lang = "en" | "fr";
type Greeting = `hello-${Lang}`;   // "hello-en" | "hello-fr"

type Route = `/users/${number}`;   // "/users/1" etc. (structural)
```

Great for API route types, event names, CSS tokens.

## 5. Union + intersection — distribution

When a conditional type is applied to a union, it **distributes** over the members:

```ts
type Nullable<T> = T | null;
type A = Nullable<string | number>;   // (string | null) | (number | null)
```

## 6. Narrowing helpers — type predicates

```ts
type Cat = { meow(): void };
type Dog = { bark(): void };

function isCat(a: Cat | Dog): a is Cat {
  return (a as Cat).meow !== undefined;
}

function speak(a: Cat | Dog) {
  if (isCat(a)) a.meow();
  else a.bark();
}
```

`a is Cat` is a **type predicate** — tells TS what the function asserts.

## 7. Discriminated unions vs inheritance

Discriminated unions are often preferred to classical inheritance in TS because:

- Pattern-match with `switch` → exhaustive, statically checked.
- No `instanceof` at runtime required.
- Flat, serializable data — plays nicely with JSON APIs.

## Interview Q&A

1. **Union vs Intersection?** — Union = "one of"; Intersection = "all of".
2. **What is a discriminated union?** — A union where each member shares a literal tag field, allowing safe narrowing via `switch`.
3. **Why `as const`?** — Prevents literal widening so types remain specific (e.g. `"N"` instead of `string`).
4. **What is a type predicate?** — A function return type `x is T` that narrows `x` after a truthy return.
5. **What is a template literal type?** — Type built from string literals + other string types using template syntax.
