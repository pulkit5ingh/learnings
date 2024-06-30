[text](https://chatgpt.com/c/0c4d90be-545a-431f-ba59-1480629eb599)

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
  6.

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
