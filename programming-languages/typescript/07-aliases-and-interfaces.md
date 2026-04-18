# 07 - Type Aliases and Interfaces

Both give a **name** to a type. Small differences decide which to use.

## Type Alias — `type`

Creates an alias for *any* type (primitive, union, intersection, tuple, function, object, mapped, conditional, ...).

```ts
type ID = string | number;
type User = { id: ID; name: string };
type Point = [number, number];
type Callback = (value: string) => void;
```

## Interface — `interface`

Describes the shape of an object (or function/class contract).

```ts
interface User {
  id: number;
  name: string;
  greet(): string;
}
```

## Extending / Combining

### `extends` — interfaces

```ts
interface Animal { name: string }
interface Dog extends Animal { breed: string }

interface Multi extends Animal, { age: number } {}   // can extend multiple
```

### Intersection `&` — type aliases

```ts
type Animal = { name: string };
type Dog = Animal & { breed: string };
```

### Interface can extend a type alias and vice versa

```ts
type Named = { name: string };
interface Person extends Named { age: number }

interface Base { id: number }
type Extra = Base & { extra: string };
```

## Declaration Merging — interface-only feature

Two interfaces with the same name are **merged**.

```ts
interface Window { customProp: string }
interface Window { another: number }
// Window now has both customProp and another
```

`type` does **not** support merging — you'll get "Duplicate identifier".

Useful for augmenting third-party type declarations (e.g. extending `Express.Request`).

## What `type` can do that `interface` can't

- Union types: `type Status = "on" | "off"`
- Tuple types: `type Pair = [string, number]`
- Mapped types: `type Readonly<T> = { readonly [K in keyof T]: T[K] }`
- Conditional types: `type NonNull<T> = T extends null | undefined ? never : T`
- Template literal types: `` type Route = `/${string}` ``

## What `interface` can do well

- Describe object and class contracts.
- Declaration merging (library augmentation).
- Natural `implements` for classes.

```ts
interface Serializable { toJSON(): string }
class User implements Serializable {
  toJSON() { return "{}"; }
}
```

## When to use which?

| Situation | Prefer |
|---|---|
| Public API / library types | `interface` (merging, cleaner errors) |
| Object / class shape | `interface` |
| Unions, tuples, mapped, conditional types | `type` |
| One-off utility aliases | `type` |

**Interview stance:** *"I default to `interface` for object and class shapes because of declaration merging and cleaner extension; I use `type` for unions, tuples, and everything the type system can compute."*

## Example — side-by-side

```ts
// Interface
interface User {
  id: number;
  name: string;
}
interface Admin extends User {
  permissions: string[];
}

// Type alias (equivalent)
type UserT = {
  id: number;
  name: string;
};
type AdminT = UserT & {
  permissions: string[];
};
```

## Gotchas

- Interface extension checks compatibility strictly; intersection (`&`) silently combines and can produce `never` for conflicting primitive props.

  ```ts
  type Bad = { x: string } & { x: number };   // x: never
  ```

- Interfaces cannot express unions directly:

  ```ts
  // interface A = string | number;   // Not allowed
  type A = string | number;
  ```

## Interview Q&A

1. **`type` vs `interface`?** — `interface` supports merging and is natural for OOP shapes; `type` is a superset — unions, tuples, mapped, conditional, template literals.
2. **What is declaration merging?** — Multiple `interface` declarations with the same name combine into one. Only interfaces can do this.
3. **Can a class `implements` a `type`?** — Yes, as long as the alias resolves to an object shape.
4. **Can an interface extend a type alias?** — Yes, if the alias is an object type.
5. **Which is better for React prop types?** — Either; `type` is common because props are often unions/intersections.
