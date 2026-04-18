# 10 - Type Casting (Type Assertions)

In TS, "casting" is really a **type assertion** — a compile-time promise, not a runtime conversion. It tells the compiler "treat this value as type X"; the emitted JS is unchanged.

## Syntax — two forms

```ts
let v: unknown = "hello";

// 1. `as` syntax (preferred; works in .tsx)
const len = (v as string).length;

// 2. Angle-bracket syntax (not allowed in .tsx — conflicts with JSX)
const len2 = (<string>v).length;
```

## Common assertions

### `as string`, `as number`, `as User`, ...

```ts
const input = document.getElementById("email") as HTMLInputElement;
input.value = "x@y.z";
```

### `as any` — escape hatch (avoid)

Turns off all type checking — fixes the error, hides the bug.

```ts
const x: any = (someValue as any);
```

### `as unknown` — safe escape hatch

When two types have no overlap, you can go via `unknown`:

```ts
const n = "42" as unknown as number;   // double assertion
```

Do this rarely, and only when you're sure of the runtime shape.

### `as const` — literal, readonly

```ts
const dirs = ["N", "S"] as const;      // readonly ["N", "S"]
const cfg  = { retries: 3 } as const;  // retries: 3, readonly
```

Primary uses:

- Derive a union from an array: `type Dir = typeof dirs[number];`
- Keep object literals narrow (e.g. Redux action types).

## Non-null assertion — `!`

Tell TS "I know this is not `null`/`undefined`".

```ts
function getLen(s?: string) {
  return s!.length;          // unsafe if s actually undefined
}
```

Prefer narrowing over `!`:

```ts
if (s) return s.length;
```

## Definite assignment — `!:`

For class fields initialized outside the constructor (e.g. by DI, lifecycle).

```ts
class C {
  svc!: Service;        // "will be assigned, trust me"
}
```

## Satisfies — `satisfies` operator (TS 4.9+)

Validates a value against a type **without widening** its inferred type.

```ts
type Palette = Record<string, string | [number, number, number]>;

const colors = {
  red: [255, 0, 0],
  blue: "#00f"
} satisfies Palette;

colors.red[0];     // number — precise tuple preserved
colors.blue.toUpperCase();   // string — precise preserved
```

Prefer `satisfies` over `as` when you want both **type-checking** *and* **preserved literal types**.

## Type guards (better than casts)

```ts
function isString(x: unknown): x is string {
  return typeof x === "string";
}

function handle(x: unknown) {
  if (isString(x)) x.toUpperCase();
}
```

Runtime check narrows the type — safer than `as`.

## DOM example

```ts
// Wrong — generic Element type has no .value
const el = document.querySelector("input");   // HTMLInputElement | null
const val = el?.value;

// Assert when you're certain:
const el2 = document.querySelector("input")! as HTMLInputElement;
```

## When casting is legitimate

- Bridging untyped data (JSON, third-party libs) after validating its shape.
- Narrowing a `unknown` after runtime check.
- Constraining a literal (`as const`).

## When to avoid

- To silence a type error without understanding it.
- To treat unrelated types as compatible (`as any`).
- When a proper guard / generic / type predicate would do.

## Interview Q&A

1. **What is a type assertion?** — Compile-time hint to the type checker; no runtime conversion.
2. **`as` vs `<T>` syntax?** — Equivalent, but `<T>` conflicts with JSX in `.tsx` files, so `as` is preferred.
3. **`as const` use cases?** — Freeze literals, derive unions from arrays, keep narrow types.
4. **`satisfies` vs `as`?** — `satisfies` enforces a type without losing literal narrowness; `as` overrides inference.
5. **What does `x!` do?** — Non-null assertion — removes `null`/`undefined` from the type (no runtime check).
