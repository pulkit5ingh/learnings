# 11 - Classes in TypeScript

TS adds access modifiers, parameter properties, abstract classes, and stronger typing on top of JS classes.

## Basic class

```ts
class User {
  id: number;
  name: string;

  constructor(id: number, name: string) {
    this.id = id;
    this.name = name;
  }

  greet(): string {
    return `Hi, ${this.name}`;
  }
}

const u = new User(1, "Pulkit");
```

## Access modifiers

| Modifier | Visible in |
|---|---|
| `public` (default) | anywhere |
| `private` | only inside the class |
| `protected` | class and subclasses |
| `readonly` | assignable only in declaration or constructor |

```ts
class Account {
  public  id: number;
  private balance: number;
  protected owner: string;
  readonly currency: string;

  constructor(id: number, balance: number, owner: string, currency: string) {
    this.id = id;
    this.balance = balance;
    this.owner = owner;
    this.currency = currency;
  }
}
```

### TS `private` vs JS `#private`

- TS `private` — compile-time only (still accessible via bracket access at runtime).
- JS `#field` — truly private at runtime. Preferred in modern code.

```ts
class Safe {
  #secret = 42;
  get() { return this.#secret; }
}
```

## Parameter properties — shorthand

Declare + assign in one go.

```ts
class User {
  constructor(
    public id: number,
    public name: string,
    private password: string
  ) {}
}
```

Equivalent to declaring fields and assigning inside the constructor.

## Getters / Setters

```ts
class Temp {
  private _c = 0;
  get celsius() { return this._c; }
  set celsius(v: number) {
    if (v < -273.15) throw new Error("below abs zero");
    this._c = v;
  }
}
```

Accessed as properties: `t.celsius = 20;`

## Inheritance — `extends`

```ts
class Animal {
  constructor(public name: string) {}
  move() { console.log(`${this.name} moves`); }
}

class Dog extends Animal {
  bark() { console.log(`${this.name} barks`); }
}
```

`super(...)` must be called in a subclass constructor before using `this`.

## `implements` — class implements interface

```ts
interface Greetable {
  greet(): string;
}

class Person implements Greetable {
  greet() { return "hi"; }
}
```

A class can `implements` multiple interfaces but only `extends` one class.

## Abstract classes

Cannot be instantiated; may contain abstract methods subclasses must implement.

```ts
abstract class Shape {
  abstract area(): number;
  describe() { return `Area is ${this.area()}`; }
}

class Circle extends Shape {
  constructor(public r: number) { super(); }
  area() { return Math.PI * this.r ** 2; }
}

// new Shape();   // Error
new Circle(3).describe();
```

## Static members

Belong to the class, not instances.

```ts
class MathUtil {
  static PI = 3.14159;
  static square(n: number) { return n * n; }
}

MathUtil.PI;
MathUtil.square(5);
```

Static fields can also be `private`, `protected`, `readonly`.

## Generic classes

```ts
class Box<T> {
  constructor(public value: T) {}
}
const b = new Box<string>("hi");
```

## Class as a type

A class name is both a **value** (constructor) and a **type** (instance shape).

```ts
class User { id!: number }
let u: User;                         // instance type
let Ctor: typeof User = User;        // constructor type
```

### Constructor signatures (when you need to type "a class")

```ts
type Constructor<T = {}> = new (...args: any[]) => T;

function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    createdAt = new Date();
  };
}
```

Classic **mixin** pattern.

## `override` keyword

With `noImplicitOverride`, methods that override a base method must be marked `override` — catches typos.

```ts
class Dog extends Animal {
  override move() { super.move(); }
}
```

## Index signature on classes

```ts
class StringMap {
  [k: string]: string;
}
```

## Common gotchas

- Forgetting to call `super()` in subclass constructor.
- Confusing `typeof Class` (constructor) with `Class` (instance).
- Arrow methods as fields vs prototype methods — arrow fields auto-bind `this` but are per-instance (more memory).
- TS `private` is not a runtime guard — use `#` for true privacy.

## Interview Q&A

1. **`public` / `private` / `protected`?** — public: anywhere; private: inside class; protected: class + subclasses.
2. **TS `private` vs JS `#field`?** — TS is compile-time only; `#` is truly private at runtime.
3. **`extends` vs `implements`?** — `extends` inherits implementation; `implements` enforces a contract (no inheritance).
4. **What is an abstract class?** — Non-instantiable base with abstract methods that subclasses must implement.
5. **Parameter properties?** — Shorthand in constructor to declare and assign class fields in one line.
6. **`typeof ClassName` vs `ClassName` as a type?** — `ClassName` is the instance type; `typeof ClassName` is the constructor type.
