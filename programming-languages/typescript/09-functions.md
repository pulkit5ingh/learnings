# 09 - Functions in TypeScript

Functions are first-class citizens; TS lets you annotate parameters, return values, overloads, and even the signature as a type.

## Basic typing

```ts
function add(a: number, b: number): number {
  return a + b;
}

const mul = (a: number, b: number): number => a * b;
```

If the return type is obvious, inference is fine. Annotate return types on **public APIs** to lock the contract.

## Optional parameters — `?`

Must come **after** required parameters.

```ts
function greet(name: string, title?: string): string {
  return title ? `${title} ${name}` : name;
}
```

`title` has type `string | undefined`.

## Default parameters

```ts
function greet(name: string, title: string = "Mr.") {
  return `${title} ${name}`;
}
```

A parameter with a default is **optional** at call sites.

## Rest parameters

```ts
function sum(...nums: number[]): number {
  return nums.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3);
```

Rest params must be a typed array (or tuple).

## `void` return

```ts
function log(msg: string): void { console.log(msg); }
```

A `void` function *can* return a value — callers just can't rely on it (useful for `.forEach` callbacks).

## `never` return

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```

Function throws or loops forever — it never returns.

## Function types

Describe a function's signature as a reusable type.

```ts
type BinaryOp = (a: number, b: number) => number;

const add: BinaryOp = (a, b) => a + b;
```

Interfaces can also describe call signatures:

```ts
interface Comparator {
  (a: number, b: number): number;
}
```

## Call, construct, and hybrid signatures

```ts
interface Counter {
  (): number;          // call signature
  count: number;       // property
  reset(): void;       // method
}
```

Use when a function also carries properties (like `jQuery` style APIs).

## Function overloads

Declare multiple signatures, one implementation. Great for APIs with argument-dependent return types.

```ts
function getValue(key: "name"): string;
function getValue(key: "age"): number;
function getValue(key: string): string | number {
  return key === "age" ? 0 : "";
}

const n = getValue("name");  // string
const a = getValue("age");   // number
```

Rules:

- Overload signatures come first; the implementation signature is **hidden** from callers.
- Keep the implementation signature broad enough to cover all overloads.

## `this` parameter

TS lets you type `this` as a **fake first parameter** (erased at runtime).

```ts
interface User { name: string }
function greet(this: User) {
  return `Hi, ${this.name}`;
}
```

`--strictBindCallApply` ensures `bind`/`call`/`apply` respect this typing.

## Arrow functions & lexical `this`

```ts
class Timer {
  seconds = 0;
  start() {
    setInterval(() => { this.seconds++; }, 1000);   // arrow keeps `this`
  }
}
```

## Generic functions (preview)

```ts
function identity<T>(x: T): T { return x; }

identity<string>("hi");
identity(42);          // inferred
```

More in `12-basic-generics.md`.

## Function type compatibility

- **Parameter bivariance / contravariance:** A function with **fewer** parameters is assignable to one with more (extra params can be ignored).
- **Return covariance:** A more specific return type is assignable to a more general one.

```ts
type Handler = (e: Event) => void;
const h: Handler = () => {};           // ok — fewer params
```

## Readonly, Immutable params

```ts
function sumAll(nums: readonly number[]): number {
  // nums.push(1);   // Error
  return nums.reduce((a, b) => a + b, 0);
}
```

## Common patterns

### Typed callbacks

```ts
function onClick(cb: (e: MouseEvent) => void) { /* ... */ }
```

### Higher-order functions

```ts
function withLog<T extends (...a: any[]) => any>(fn: T): T {
  return ((...args: Parameters<T>) => {
    console.log("call", args);
    return fn(...args);
  }) as T;
}
```

## Interview Q&A

1. **Optional vs default parameters?** — Optional means "may be `undefined`"; default means "has a fallback value" — both make it optional at call site.
2. **When to use overloads?** — When return type depends on argument value/type, and a union return would be lossy.
3. **`void` vs `undefined` return?** — `void` says the caller shouldn't use the return; `undefined` requires an actual `undefined`.
4. **Why does `void` allow returning a value?** — For compatibility with callbacks whose return is ignored (e.g. `Array.forEach`).
5. **What is the `this` parameter?** — A purely-typed first parameter (erased in JS) that constrains how a function may be called.
