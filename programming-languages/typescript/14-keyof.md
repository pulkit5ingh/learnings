# 14 - `keyof`, `typeof`, Mapped & Conditional Types

These four features power most of TS's advanced type-level programming.

## 1. `keyof` — the union of a type's keys

```ts
interface User { id: number; name: string }
type UserKeys = keyof User;        // "id" | "name"
```

### Typed property access

```ts
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const u = { id: 1, name: "a" };
getProp(u, "id");        // number
// getProp(u, "nope");   // Error
```

`T[K]` is an **indexed access type** — the type of `T`'s property `K`.

### `keyof` with index signatures

```ts
type StrMap = { [k: string]: number };
type K = keyof StrMap;     // string | number (JS coerces numeric keys)

type NumMap = { [k: number]: string };
type K2 = keyof NumMap;    // number
```

## 2. `typeof` — type of a value

In a **type position**, `typeof` pulls a type from a runtime value.

```ts
const config = { retries: 3, url: "/api" };
type Config = typeof config;         // { retries: number; url: string }

function add(a: number, b: number) { return a + b; }
type Add = typeof add;               // (a: number, b: number) => number
```

### Combine with `keyof`

```ts
const roles = ["admin", "user", "guest"] as const;
type Role = typeof roles[number];    // "admin" | "user" | "guest"

const actions = { save() {}, cancel() {} };
type ActionName = keyof typeof actions;   // "save" | "cancel"
```

## 3. Indexed access types — `T[K]`

```ts
type User = { id: number; addr: { city: string } };

type Id = User["id"];             // number
type City = User["addr"]["city"]; // string

type Values = User[keyof User];   // number | { city: string }
```

Works on arrays too:

```ts
type Names = string[];
type Name = Names[number];        // string
```

## 4. Mapped types — transform keys

Shape: `{ [K in Keys]: ValueType }`.

```ts
type Stringify<T> = { [K in keyof T]: string };

type S = Stringify<{ a: number; b: boolean }>;
// { a: string; b: string }
```

### With modifiers — `?`, `readonly`, `+`, `-`

```ts
type MyPartial<T>  = { [K in keyof T]?: T[K] };          // add ?
type MyRequired<T> = { [K in keyof T]-?: T[K] };         // remove ?
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };  // add readonly
type Writable<T>   = { -readonly [K in keyof T]: T[K] }; // remove readonly
```

### Key remapping — `as` clause (TS 4.1+)

```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

type G = Getters<{ name: string; age: number }>;
// { getName: () => string; getAge: () => number }
```

Filter keys with `never`:

```ts
type PickString<T> = {
  [K in keyof T as T[K] extends string ? K : never]: T[K]
};
```

## 5. Conditional types — `T extends U ? X : Y`

Type-level `if`.

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<"x">;     // true
type B = IsString<42>;      // false
```

### `infer` — capture a type in the condition

```ts
type ElementType<T> = T extends (infer U)[] ? U : T;

type E1 = ElementType<number[]>;   // number
type E2 = ElementType<string>;     // string
```

Implement your own utility types:

```ts
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;
type Awaited2<T>     = T extends Promise<infer U> ? Awaited2<U> : T;
```

### Distribution over unions

Naked conditional types **distribute** over unions.

```ts
type NonNull<T> = T extends null | undefined ? never : T;
type R = NonNull<string | null | number>;   // string | number
```

To **prevent** distribution, wrap in a tuple:

```ts
type IsUnion<T> =
  [T] extends [never] ? false :
  // ...
  true;
```

## 6. Template literal types (combine with mapped/conditional)

```ts
type EventName<T extends string> = `on${Capitalize<T>}`;
type E = EventName<"click" | "focus">;   // "onClick" | "onFocus"
```

## 7. Putting it together — real examples

### Typed event emitter

```ts
type Events = {
  login: { user: string };
  logout: void;
};

function emit<K extends keyof Events>(event: K, payload: Events[K]) {}

emit("login", { user: "x" });   // ok
// emit("login", null);         // Error
```

### Dot-path autocompletion (advanced)

```ts
type Path<T, Prev extends string = ""> =
  T extends object
    ? { [K in keyof T & string]: Prev extends "" ? K | Path<T[K], K> : `${Prev}.${K}` | Path<T[K], `${Prev}.${K}`> }[keyof T & string]
    : never;
```

(Heavy but popular pattern for object-path libraries.)

## Interview Q&A

1. **What does `keyof T` give you?** — A union of `T`'s property names.
2. **What does `typeof value` do in a type position?** — Captures the type of a runtime value.
3. **`typeof arr[number]`?** — Converts a `readonly` literal array into a union of its element types.
4. **What is a mapped type?** — A type built by iterating keys: `{ [K in K] : T }`.
5. **What is `infer`?** — Within a conditional type, captures a type for use in the true branch.
6. **What is distribution in conditional types?** — Naked `T extends U ? X : Y` distributes over a union `T`, applying per-member.
