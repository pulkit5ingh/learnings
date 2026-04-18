# 02 - Types in TypeScript

TypeScript's type system can be split into three buckets:

1. **Primitive types** — `string`, `number`, `boolean`, `bigint`, `symbol`, `null`, `undefined`, `void`.
2. **Object types** — arrays, tuples, objects, functions, classes, interfaces, enums.
3. **Special / Top-Bottom types** — `any`, `unknown`, `never`, `object`.

## 1. Type Annotation vs Type Inference

```ts
let a: number = 10;      // explicit annotation
let b = 10;              // inferred as number
```

- Prefer **inference** when obvious.
- Use **annotations** on function signatures, public APIs, and when inference isn't enough.

## 2. Primitive Types

```ts
let isDone: boolean = false;
let age: number = 30;                  // covers int, float, hex, binary
let firstName: string = "Pulkit";
let big: bigint = 9007199254740991n;
let sym: symbol = Symbol("id");
let u: undefined = undefined;
let n: null = null;
```

- `number` covers integers & floats (no separate `int`/`float`).
- `bigint` for arbitrarily large integers (ES2020+).
- `symbol` for unique keys.

### `void`

Return type of functions that don't return a value.

```ts
function log(msg: string): void {
  console.log(msg);
}
```

### `null` and `undefined`

- With `strictNullChecks: true`, `null` and `undefined` are their own types and are **not** assignable to `string`, `number`, etc. — must be in a union.

```ts
let name: string | null = null;
```

## 3. Object Types

```ts
let user: { id: number; name: string } = { id: 1, name: "Pulkit" };
```

Sub-categories (covered later):

- **Interface** — named object shape (`interface User { id: number }`).
- **Class** — blueprint that also creates a type.
- **Enum** — named set of constants.
- **Array** — `number[]` or `Array<number>`.
- **Tuple** — fixed-length, fixed-type array: `[string, number]`.

## 4. Special / Top-Bottom Types

### `any` — disables type checking (escape hatch)

```ts
let data: any = 10;
data = "hi";    // ok
data.foo.bar;   // ok (but unsafe)
```

- Avoid when possible; treats the value as untyped.
- Commonly sneaks in via untyped JS libs.

### `unknown` — type-safe `any`

```ts
let val: unknown = "hi";
// val.toUpperCase();       // Error — must narrow first
if (typeof val === "string") {
  val.toUpperCase();        // ok, narrowed
}
```

- You **must narrow** `unknown` before using it.
- Prefer `unknown` over `any` at API boundaries (JSON parse, fetch responses).

### `never` — value that never occurs

- Function that throws or runs forever.
- Exhaustiveness checks in `switch`.

```ts
function fail(msg: string): never { throw new Error(msg); }

type Shape = { kind: "circle" } | { kind: "square" };
function area(s: Shape) {
  switch (s.kind) {
    case "circle": return 0;
    case "square": return 0;
    default:
      const _exhaustive: never = s;   // error if new shape added
      return _exhaustive;
  }
}
```

### `object` — any non-primitive

```ts
let o: object = { a: 1 };
// o = 1;   // Error
```

Less useful than `Record<string, unknown>` or a precise shape.

## 5. Type Assertion (Casting)

Tell the compiler "trust me, this is of type X". Does **not** change runtime value.

```ts
let val: unknown = "hello";
let len1 = (val as string).length;       // preferred
let len2 = (<string>val).length;         // JSX-incompatible syntax

// as const — literal, readonly
const dirs = ["N", "S"] as const;        // readonly ["N", "S"]

// double assertion (escape hatch — avoid)
const x = (someValue as unknown) as TargetType;
```

- `as` is compile-time only — no runtime check.
- Use `unknown` + narrowing instead when possible.

## 6. Type Hierarchy (mental model)

```
            unknown  (top — accepts everything)
           /   |   \
        any    object  primitives
           \   |   /
             never    (bottom — assignable to everything)
```

- `any` turns off type checking in both directions.
- `unknown` accepts everything but *produces* nothing without narrowing.
- `never` is assignable to every type; nothing is assignable to `never` (except `never`).

## Interview Q&A

1. **`any` vs `unknown`?** — `any` opts out of type checking; `unknown` forces narrowing before use.
2. **What is `never`?** — A type for values that never occur (throws / infinite loops); used for exhaustiveness checks.
3. **`void` vs `undefined`?** — `void` = function returns nothing meaningful; `undefined` = the specific value.
4. **What does `as const` do?** — Widens literals to readonly literal types (arrays become readonly tuples).
5. **Is type assertion a runtime cast?** — No. It's purely compile-time. No conversion happens.
