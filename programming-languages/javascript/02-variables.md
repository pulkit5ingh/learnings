# Differences Between `var`, `let`, and `const`

## **1. Overview of `var`, `let`, and `const`**

| **Feature**        | **`var`**                              | **`let`**                              | **`const`**                                    |
| ------------------ | -------------------------------------- | -------------------------------------- | ---------------------------------------------- |
| **Scope**          | Function-scoped                        | Block-scoped                           | Block-scoped                                   |
| **Re-declaration** | Allowed within the same scope          | Not allowed in the same scope          | Not allowed in the same scope                  |
| **Re-assignment**  | Allowed                                | Allowed                                | Not allowed after initialization               |
| **Initialization** | Can be declared without initialization | Can be declared without initialization | Must be initialized at the time of declaration |
| **Hoisting**       | Hoisted but initialized to `undefined` | Hoisted but not initialized            | Hoisted but not initialized                    |

---

## **2. Scope Differences**

| **Type**            | **Function Scope**                              | **Block Scope**                                                                                                             |
| ------------------- | ----------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| **`var`**           | Variables are scoped to the enclosing function. | Does not support block scope. A `var` inside a block (`if`, `for`, etc.) leaks into the enclosing function or global scope. |
| **`let` / `const`** | Not function-scoped.                            | Variables are confined to the block they are declared in.                                                                   |

**Example:**

```javascript
if (true) {
  var x = 10;
  let y = 20;
  const z = 30;
}
console.log(x); // 10 (accessible because of function scope)
console.log(y); // ReferenceError (block-scoped)
console.log(z); // ReferenceError (block-scoped)
```

---

## **3. Re-declaration and Re-assignment**

| **Feature**        | **`var`** | **`let`**   | **`const`** |
| ------------------ | --------- | ----------- | ----------- |
| **Re-declaration** | Allowed   | Not allowed | Not allowed |
| **Re-assignment**  | Allowed   | Allowed     | Not allowed |

**Example:**

```javascript
// Re-declaration
var a = 10;
var a = 20; // Allowed

let b = 30;
// let b = 40; // SyntaxError: Identifier 'b' has already been declared

const c = 50;
// const c = 60; // SyntaxError: Identifier 'c' has already been declared

// Re-assignment
a = 15; // Allowed
b = 35; // Allowed
// c = 55; // TypeError: Assignment to constant variable
```

---

## **4. Hoisting Behavior**

| **Type**    | **Hoisting Behavior**                                                                   |
| ----------- | --------------------------------------------------------------------------------------- |
| **`var`**   | Hoisted to the top of the scope but initialized as `undefined`.                         |
| **`let`**   | Hoisted but not initialized. Accessing it before declaration causes a `ReferenceError`. |
| **`const`** | Hoisted but not initialized. Must be initialized at the time of declaration.            |

**Example:**

```javascript
console.log(a); // undefined (hoisted)
var a = 10;

console.log(b); // ReferenceError (temporal dead zone)
let b = 20;

console.log(c); // ReferenceError (temporal dead zone)
const c = 30;
```

---

## **5. Similarities Between `let` and `const`**

| **Feature**                    | **`let`** and **`const`**                                      |
| ------------------------------ | -------------------------------------------------------------- |
| **Block Scope**                | Both are confined to the block in which they are declared.     |
| **No Hoisting Initialization** | Both are hoisted but cannot be accessed before initialization. |
| **Better Practice**            | Preferred over `var` for predictable scoping.                  |

---

## **6. When to Use Which?**

| **Scenario**                                   | **Recommended Keyword**                                   |
| ---------------------------------------------- | --------------------------------------------------------- |
| **Re-declare variables or use function scope** | `var` (generally avoid unless necessary for legacy code). |
| **Variables that may change**                  | `let` (e.g., counters, flags, intermediate calculations). |
| **Variables that should not change**           | `const` (e.g., configuration settings, fixed values).     |

---

## **7. Explanation of Hoisting**

### **What is Hoisting?**

Hoisting is JavaScript's default behavior of moving declarations to the top of their scope during the compile phase.

- **`var`**: Hoisted and initialized to `undefined`.
- **`let` / `const`**: Hoisted but not initialized. This creates a **temporal dead zone (TDZ)** from the start of the block until the declaration is encountered.

---

### **Why Hoisting Works This Way?**

1. **Compilation Phase**:
   JavaScript first scans the code to create a memory space for variable and function declarations. At this stage:

   - `var` variables are initialized to `undefined`.
   - `let` and `const` variables are "hoisted" but left uninitialized, hence the TDZ.
   - Function declarations are fully hoisted.

2. **Execution Phase**:
   JavaScript starts executing code line by line. Variables and functions are assigned values during this phase.

---

## **8. Summary of Hoisting**

| **Type**    | **Hoisting**                            | **Access Before Declaration**     |
| ----------- | --------------------------------------- | --------------------------------- |
| **`var`**   | Hoisted and initialized to `undefined`. | Allowed but value is `undefined`. |
| **`let`**   | Hoisted but not initialized.            | Causes a `ReferenceError`.        |
| **`const`** | Hoisted but not initialized.            | Causes a `ReferenceError`.        |

**Example:**

```javascript
// var
console.log(foo); // undefined
var foo = 10;

// let
console.log(bar); // ReferenceError
let bar = 20;

// const
console.log(baz); // ReferenceError
const baz = 30;
```

---

## **Conclusion**

1. Use **`const`** whenever possible for variables that do not need reassignment.
2. Use **`let`** for variables that need to be reassigned within the same scope.
3. Avoid **`var`** unless working with legacy code or requiring function-scoped behavior.
