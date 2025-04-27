1. what is typescript ? how it is different from javascript ?

- Typescript is a strongly typed superset of javascript that adds optional static types, interfaces and other features. It compiles to plain javascript.
- Some of the key differences are :
- - Static typing.
- - Interfaces and enums.
- - Advancing tolling with IDEs due to type safety.

2. What are the advantages of using typescript ?

- Static typing : Catch error at compile time.
- Better tooling : Enhanced auto-completion, refactoring, and debugging support.
- Code scalability: Easier to manage large projects.
- Advanced features: Interfaces, Generics, enums etc...

3. What are some basic types in typescript ?

- Some of the basic types in typescript are string, number, boolean, null, undefined, any, void, array, tuple, enum, unknown, never.

```typescript
let name: string = 'John doe';
let age: number = 45;
let height: number = 5.9;
let isStudent: boolean = true;
let data: null = null;
let result: undefined = undefined;
let randomValue: any = 'Hello';
randomValue = 42; // No error
function logMessage(message: string): void {
  console.log(message);
}
let numbers: number[] = [1, 2, 3];
let names: string[] = ['Alice', 'Bob', 'Charlie'];
let person: [string, number] = ['Alice', 25];
enum Color {
  Red,
  Green,
  Blue,
}
let favoriteColor: Color = Color.Green;
let userInput: unknown = 'Hello';
if (typeof userInput === 'string') {
  console.log(userInput.toUpperCase());
}
function throwError(message: string): never {
  throw new Error(message);
}
```

4. what are the interfaces in typescript & how are they used ?

- Interface defines the shape of an object.

```typescript
interface User {
  id: number;
  name: string;
  email?: string; // Optional property
}

const user: User = { id: 1, name: 'John' };
```

5. what are generic in typescript ?

- Generics allow you to create reusable and flexible components or functions that can work with any data type.

```typescript
function identity<T>(arg: T): T {
  return arg;
}
const num = identity<number>(42);
const str = identity<string>('Hello');
```

6. Explain any, unknown, never, and void in TypeScript.

- any: Opts out of type checking. Can store any value.
  unknown: Safer alternative to any; requires type assertions or checks before use.
  never: Represents the type of values that never occur (e.g., functions that throw errors or infinite loops).
  void: Represents the absence of a return value (e.g., for functions that don’t return anything).

7. How does TypeScript handle modules?

- TypeScript supports ES modules and CommonJS modules. You can export and import:

```typescript
// File: utils.ts
export function add(a: number, b: number): number {
  return a + b;
}

// File: main.ts
import { add } from './utils';
console.log(add(2, 3));
```

8. What are TypeScript decorators?

- Decorators are special annotations for modifying classes, methods, or properties.

```typescript
function Log(target: any, propertyKey: string) {
  console.log(`Property ${propertyKey} accessed`);
}

class Example {
  @Log
  name: string;
}
```

9. What is the readonly modifier in TypeScript?

- Ensures that a property cannot be modified after initialization.

```typescript
class Point {
  readonly x: number = 10;
  readonly y: number;

  constructor(y: number) {
    this.y = y;
  }
}

const point = new Point(20);
// point.x = 15; // Error
```

10. What is the difference between interface and type?

- interface: Used to define object shapes and is extendable.
- type: More versatile; can represent primitive types, union types, and intersections.

```typescript
interface A {
  x: number;
}
type B = { y: number };
```

11.
