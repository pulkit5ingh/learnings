# 03 - Arrays in TypeScript

An array is an ordered, **homogeneous** collection (same element type). Two equivalent syntaxes:

```ts
let nums: number[] = [1, 2, 3];
let names: Array<string> = ["a", "b"];    // generic form
```

Prefer `T[]` for simple cases; use `Array<T>` when the element type itself contains `|` or is complex.

## Declaring Arrays

```ts
let empty: number[] = [];
let mixed: (string | number)[] = ["a", 1];   // union of types
let matrix: number[][] = [[1, 2], [3, 4]];   // 2D
```

## Type Inference

```ts
const arr = [1, 2, 3];          // inferred number[]
const arr2 = [1, "a"];          // inferred (string | number)[]
```

## `readonly` Arrays

Prevent mutation after creation.

```ts
const xs: readonly number[] = [1, 2, 3];
// xs.push(4);     // Error
// xs[0] = 9;      // Error

// Equivalent generic form
const ys: ReadonlyArray<number> = [1, 2, 3];
```

`readonly` only blocks mutation through *that* reference — not a deep freeze.

## `as const` — literal & readonly

```ts
const roles = ["admin", "user", "guest"] as const;
// type: readonly ["admin", "user", "guest"]

type Role = typeof roles[number];   // "admin" | "user" | "guest"
```

Great for deriving a union type from a list.

## Iteration

```ts
const nums = [1, 2, 3];

for (const n of nums) console.log(n);            // values
for (const i in nums) console.log(i);            // indices (as string) — avoid for arrays
nums.forEach((n, i) => console.log(i, n));
const doubled = nums.map(n => n * 2);            // number[]
const evens = nums.filter(n => n % 2 === 0);
const sum = nums.reduce((acc, n) => acc + n, 0); // number
```

Array methods preserve element type via generics on `Array<T>`.

## Common built-in methods (typed)

| Method | Returns |
|---|---|
| `map<U>(fn): U[]` | transformed array |
| `filter(fn): T[]` | same-typed subset |
| `reduce<U>(fn, init): U` | single value |
| `find(fn): T \| undefined` | first match |
| `some` / `every` | `boolean` |
| `includes(v): boolean` | membership |
| `sort(fn?): this` | mutates & returns |
| `concat(...): T[]` | merged |
| `slice(s, e): T[]` | shallow copy |
| `flat<D>(): ...` | depth-typed flatten |

## Spread & Rest with arrays

```ts
const a = [1, 2];
const b = [...a, 3, 4];                 // number[]

function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}
```

## `Array<T>` vs Tuple

- Array: variable length, one element type (or union).
- Tuple: **fixed length**, positional types (next file).

```ts
const arr: number[] = [1, 2, 3, 4];       // length varies
const tup: [string, number] = ["a", 1];   // exactly 2 elems
```

## Array of objects

```ts
interface User { id: number; name: string; }
const users: User[] = [
  { id: 1, name: "A" },
  { id: 2, name: "B" },
];
```

## `Array.from` and typed generics

```ts
const chars = Array.from("hello");        // string[]
const zeros = Array.from<number>({ length: 3 }, () => 0); // number[]
```

## Interview Q&A

1. **`T[]` vs `Array<T>`?** — Same thing. Stylistic preference; `Array<T>` required in some generic contexts.
2. **What does `readonly T[]` do?** — Prevents mutation methods (`push`, `splice`, index assignment) via the reference.
3. **How to derive a union from an array?** — `typeof arr[number]` with `as const`.
4. **`for...in` vs `for...of`?** — `for...in` yields keys (indices as strings, incl. inherited); `for...of` yields values. Use `for...of` for arrays.
5. **Can arrays hold mixed types?** — Yes, via a union: `(string | number)[]`.
