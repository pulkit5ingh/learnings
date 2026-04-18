# 06 - Enums in TypeScript

An **enum** is a named set of related constants. Helps replace magic numbers/strings with readable identifiers.

```ts
enum Direction { Up, Down, Left, Right }

const d: Direction = Direction.Up;
```

## 1. Numeric enums (default)

Auto-increments from `0`.

```ts
enum Status {
  Pending,     // 0
  Active,      // 1
  Closed       // 2
}

Status.Active;        // 1
Status[1];            // "Active"  ← reverse mapping
```

You can initialize values manually:

```ts
enum HttpCode {
  OK = 200,
  Created = 201,
  BadRequest = 400,
  NotFound = 404
}
```

**Reverse mapping** (number ↔ name) is a feature of *numeric* enums only.

## 2. String enums

Each member has an explicit string value.

```ts
enum Role {
  Admin = "ADMIN",
  User  = "USER",
  Guest = "GUEST"
}
```

- **No reverse mapping** (only one-way).
- Preferred for serialization, logging, API payloads — values are stable and self-descriptive.

## 3. Heterogeneous enums (rare / avoid)

Mix strings and numbers. Legal but not recommended.

```ts
enum Mixed {
  No = 0,
  Yes = "YES"
}
```

## 4. Const enums

Compiled away — members are inlined into the call sites. Smallest output, fastest at runtime.

```ts
const enum Colors { Red, Green, Blue }
const c = Colors.Red;   // emitted as: const c = 0 /* Red */;
```

Limitations:

- Cannot be used with `isolatedModules` unless carefully configured.
- Can't be iterated or accessed dynamically at runtime (no object exists).

## 5. Computed & constant members

```ts
enum FileAccess {
  None = 0,
  Read = 1 << 1,           // constant expression
  Write = 1 << 2,
  ReadWrite = Read | Write,
  G = "123".length          // computed
}
```

Rule: after a computed member, subsequent members need initializers.

## 6. Enum as type and as value

An enum creates **both** a runtime object and a type.

```ts
enum Role { Admin, User }

function setRole(r: Role) { /* ... */ }   // Role used as a type
setRole(Role.Admin);                      // Role used as a value
```

## 7. Enum vs union of literals

Modern TS often prefers a **union of string literals** over string enums.

```ts
type Role = "admin" | "user" | "guest";

const r: Role = "admin";
```

| | String enum | Literal union |
|---|---|---|
| Runtime footprint | Yes (emits an object) | No |
| `import type` friendly | No | Yes |
| Exhaustiveness in `switch` | Yes | Yes |
| Value list to iterate | Yes (`Object.values(Role)`) | No (unless using `as const` array) |
| Autocomplete | Yes | Yes |

**Interview stance:** *"I use string literal unions by default for zero runtime cost; I reach for enums when I need a shared, iterable, named set of values."*

## 8. Common usage patterns

### With `switch`

```ts
enum Shape { Circle, Square }

function area(s: Shape, size: number) {
  switch (s) {
    case Shape.Circle: return Math.PI * size ** 2;
    case Shape.Square: return size * size;
    default: {
      const _: never = s;   // exhaustiveness check
      return _;
    }
  }
}
```

### Iterating a numeric enum

```ts
enum Status { Pending, Active, Closed }

Object.values(Status)
  .filter(v => typeof v === "number");   // [0, 1, 2]
```

### Iterating a string enum

```ts
enum Role { Admin = "ADMIN", User = "USER" }
Object.values(Role);   // ["ADMIN", "USER"]
```

## Interview Q&A

1. **Numeric vs string enum?** — Numeric auto-increments and has reverse mapping; string enums are explicit, one-way, and safer for serialization.
2. **What is a const enum?** — Inlined at compile time; no runtime object — smallest output.
3. **Enum vs union of literals?** — Unions have zero runtime cost; enums create a runtime object but give a shared symbol and iterability.
4. **Does a string enum have reverse mapping?** — No, only numeric enums do.
5. **Why avoid heterogeneous enums?** — Confusing and rarely meaningful.
