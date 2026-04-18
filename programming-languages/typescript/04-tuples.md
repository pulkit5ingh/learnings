# 04 - Tuples in TypeScript

A **tuple** is an array with a **fixed length** and **fixed types at each position**. Unlike `T[]`, both order and size are part of the type.

```ts
let user: [number, string] = [1, "Pulkit"];
// user = ["Pulkit", 1];   // Error — wrong order
// user = [1];             // Error — wrong length
```

## Why tuples?

- Return multiple values from a function (like React's `useState`).
- Represent records where position has meaning (e.g. `[lat, lng]`).
- Stronger guarantees than `any[]`.

## Declaring tuples

```ts
let point: [number, number] = [10, 20];
let row: [string, number, boolean] = ["Pulkit", 25, true];
```

## Optional tuple elements

```ts
let t: [string, number?] = ["a"];
t = ["a", 1];   // also ok
```

## Rest elements in tuples

A tuple can contain a rest element to represent variable-length suffixes/prefixes.

```ts
type StringsThenNumbers = [...string[], number];
type NumberThenStrings  = [number, ...string[]];

const a: StringsThenNumbers = ["a", "b", 10];
const b: NumberThenStrings  = [10, "x", "y"];
```

## Readonly tuples

```ts
const point: readonly [number, number] = [10, 20];
// point[0] = 1;   // Error
```

`as const` turns a literal array into a readonly tuple:

```ts
const pair = [1, "a"] as const;   // readonly [1, "a"]
```

## Named tuples (labels)

Labels improve readability — they're purely for docs/IntelliSense.

```ts
type Range = [start: number, end: number];
const r: Range = [0, 10];
```

## Tuples as function returns

Classic interview pattern — mimicking `useState`.

```ts
function useState<T>(initial: T): [T, (v: T) => void] {
  let value = initial;
  const set = (v: T) => { value = v; };
  return [value, set];
}

const [count, setCount] = useState(0);   // count: number
```

## Tuple vs Array — quick contrast

| Aspect | Tuple | Array |
|---|---|---|
| Length | Fixed | Variable |
| Types | Per position | Single (or union) |
| Use case | Heterogeneous, positional | Homogeneous collection |
| Destructuring | Typed per slot | Single element type |

## Gotchas

- Tuples are still JS arrays at runtime — `length`, `push`, index access all work unless marked `readonly`.
- Without `readonly`, you can technically `push` to a tuple and TS allows it:

  ```ts
  const t: [string, number] = ["a", 1];
  t.push("b");   // compiles but breaks the invariant
  ```

  Use `readonly [...]` to prevent this.

## Interview Q&A

1. **Array vs Tuple?** — Array is variable-length/single-type; tuple is fixed-length with positional types.
2. **Can a tuple be readonly?** — Yes: `readonly [T1, T2]` or via `as const`.
3. **What is a rest element in a tuple?** — `[T, ...U[]]` represents a head plus a variable-length tail.
4. **Where are tuples commonly used?** — React hooks (`useState`), coordinate pairs, fixed-format records.
5. **Do named tuple labels affect runtime?** — No, purely for documentation/IntelliSense.
