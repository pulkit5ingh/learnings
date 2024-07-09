[text](https://chatgpt.com/c/0c4d90be-545a-431f-ba59-1480629eb599)

```javascript
_____________________________________________________________________
|                | var                  | const     | let           |
|----------------|----------------------|-----------|---------------|
| scope          | global or functional | block     | block         |
| redeclare?     | yes                  | no        | no            |
| reassign?      | yes                  | no        | yes           |
| hoisted?       | yes                  | no        | no            |
|___________________________________________________________________|

```

### Variable declarations

- JavaScript Variables can be declared in 4 ways:

- Automatically Using var
- Using let
- Using const

- When to Use var, let, or const?

  1. Always declare variables
  2. Always use const if the value should not be changed
  3. Always use const if the type should not be changed (Arrays and Objects)
  4. Only use let if you can't use const
  5. Only use var if you MUST support old browsers.

### Variable naming rules

- All JavaScript variables must be identified with unique names.
- These unique names are called identifiers.
- Identifiers can be short names (like x and y) or more descriptive names (age, sum, totalVolume).
- The general rules for constructing names for variables (unique identifiers) are:
  1. Names can contain letters, digits, underscores, and dollar signs.
  2. Names must begin with a letter.
  3. Names can also begin with $ and \_ (but we will not use it in this tutorial).
  4. Names are case sensitive (y and Y are different variables).
  5. Reserved words (like JavaScript keywords) cannot be used as names.

### Variable scopes

- Scope determines the accessibility (visibility) of variables.
- JavaScript variables have 3 types of scope:

- Global scope
- Block scope
- Function scope

- Global scope

```javascript
function varScope() {
  if (true) {
    var x = 10;
    console.log(x); // 10
  }
  console.log(x); // 10 (still accessible outside the block)
}
varScope();
console.log(typeof x); // undefined (x is not accessible outside the function)

function letScope() {
  if (true) {
    let y = 20;
    console.log(y); // 20
  }
  // console.log(y); // ReferenceError: y is not defined
}
letScope();
console.log(typeof y); // undefined (y is not accessible outside the function)

function constScope() {
  if (true) {
    const z = 30;
    console.log(z); // 30
  }
  // console.log(z); // ReferenceError: z is not defined
}
constScope();
console.log(typeof z); // undefined (z is not accessible outside the function)
```

- Block scope

```javascript
{
  let x = 10;
  console.log(x); // 10
}
// console.log(x); // Uncaught ReferenceError: x is not defined
{
  const y = 20;
  console.log(y); // 20
}
// console.log(y); // Uncaught ReferenceError: y is not defined
{
  const y = 20;
  console.log(y); // 20
}
// console.log(y); // Uncaught ReferenceError: y is not defined
```

### Hoisting

```javascript

Hoisting in JavaScript is a behavior in which variable and function declarations are moved (or "hoisted") to the top of their containing scope during the compilation phase, before the code is executed. This means that you can use variables and functions before they are declared in the code.

- Variable Hoisting
In the case of variables, only the declaration is hoisted, not the initialization. If you try to use a variable before it's declared and initialized, you'll get undefined.

console.log(x); // undefined
var x = 5;
console.log(x); // 5

- Function Hoisting
Function declarations are fully hoisted, meaning both the declaration and the body of the function are moved to the top of the scope.

console.log(foo()); // "Hello"
function foo() {
  return "Hello";
}

However, function expressions are not hoisted. If you try to use a function expression before it's defined, you'll get an error.

console.log(bar()); // TypeError: bar is not a function
var bar = function() {
  return "Hello";
};

- Let and Const
Variables declared with let and const are also hoisted, but they are not initialized. Accessing them before the declaration results in a ReferenceError.

console.log(a); // ReferenceError: Cannot access 'a' before initialization
let a = 3;

console.log(b); // ReferenceError: Cannot access 'b' before initialization
const b = 5;

This is because let and const declarations are hoisted to the top of their block scope but are in a "temporal dead zone" from the start of the block until the declaration is encountered.

```
