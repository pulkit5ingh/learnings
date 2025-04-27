```js
create a a heading for this topic and a summarized content for it with some example easy to understand and learn, at the end add 20 practical question and answer for this topic so that student can grab strong understand of it.
response should be in .md format in a single file. so that i can copy and paste it in my vs code editor.
```

## 📚 Introduction

## Javascript code execution Overview

- JS is single-threaded and synchronous.
- Components: Call Stack, Heap, Event Loop.
- Phases: Creation → Execution.
- Concepts: Execution Context, Memory Heap.

---

## 🎯 Key Concepts

- **JavaScript is a synchronous, single-threaded language**.
  - **Synchronous**: Executes one task at a time in a specific order.
  - **Single-threaded**: Only **one line of code** can be executed at any moment.

> ❓ What about `asynchronous` like AJAX?
> ➡️ That involves **other concepts** like **Web APIs**, **Callback Queue**, and **Event Loop**, which we’ll learn separately.

---

## 🧩 Quick Example

```js
console.log('1');
console.log('2');
console.log('3');
```

Output:

```js
1;
2;
3;
```

Runs top to bottom in one thread.

---

## 🧠 What is an Execution Context?

The Execution Context has **two main components**:

1. **Memory Component (Variable Environment)**

   - All **variables** and **functions** are stored here as **key-value pairs**.
   - Example:
     ```javascript
     var a = 10;
     function greet() {
       console.log('Hello');
     }
     ```
     Memory stores:
     ```
     a: 10
     greet: function() { console.log("Hello"); }
     ```

2. **Code Component (Thread of Execution)**
   - The place where code is **executed one line at a time**.
   - It's like a thread moving line-by-line through your code.

---

## Thread of Execution

- The flow in which code executes line-by-line.

---

## JS Code Execution Flow

1. Global Execution Context Created
2. Memory Creation Phase:
   - Variables hoisted (var → undefined)
   - Functions hoisted fully
3. Execution Phase:
   - Code runs line-by-line

---

## Memory Environment & Code Component

- **Memory/Variable Environment:** Holds vars/functions
- **Code Component (Thread):** Executes logic

---

## 📝 Recap

- Execution Context : Big box where JS code runs.
- Memory Component : Stores variables/functions (variable environment).
- Code Component : Executes code line-by-line (thread of execution).
- Synchronous : One task at a time, in order.
- Single-threaded : Only one line running at a time.

## ✅ 20 Practical Questions & Answers

# 🧠 JavaScript Execution Context - Quiz

Welcome to your mini test on Execution Context!
Try to answer each question yourself first before checking the answers! 🚀

---

### 1. What is an Execution Context in JavaScript?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
Execution Context is the environment where JavaScript code is evaluated and executed.
</details>

---

### 2. Name the two main parts of an Execution Context.

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
Memory Component (Variable Environment) and Code Component (Thread of Execution).
</details>

---

### 3. What does the Memory Component store?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
Variables and functions as key-value pairs.
</details>

---

### 4. Another name for the Memory Component is?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
Variable Environment.
</details>

---

### 5. What happens inside the Code Component?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
Code is executed line-by-line.
</details>

---

### 6. What is another name for the Code Component?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
Thread of Execution.
</details>

---

### 7. Is JavaScript single-threaded or multi-threaded?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
Single-threaded.
</details>

---

### 8. What does "synchronous" mean in JavaScript?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
JavaScript executes one command at a time in a specific order.
</details>

---

### 9. Can JavaScript execute multiple lines of code at the same time?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
No, only one line at a time.
</details>

---

### 10. What happens when a function is declared?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
It is stored in the Memory Component.
</details>

---

### 11. What will be the initial value of variables during the memory allocation phase?

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
`undefined`.
</details>

---

### 12. Define "Thread of Execution" in simple terms.

➡️ _Your Answer:_

<details>
<summary>Answer</summary>
The step-by-step execution of JavaScript code, line-by-line.
</details>

---

### 13. What will be the output of the following code?

```javascript
console.log(x);
var x = 10;
```

<details> <summary>Answer</summary> `undefined` (due to hoisting). </details>

---

### 14. What happens if you try to access a variable before declaring it?

➡️ _Your Answer:_

## <details> <summary>Answer</summary> It returns `undefined` due to hoisting. </details>

---

### 15. Why is JavaScript called a single-threaded language?

➡️ _Your Answer:_

<details> <summary>Answer</summary> Because it can execute only one command at a time using a single call stack. </details>

---

### 16. How does memory allocation happen before execution starts?

➡️ _Your Answer:_

<details> <summary>Answer</summary> All variables and functions are allocated memory with default values like `undefined`. </details>

---

### 17. When does JavaScript start executing actual code?

➡️ _Your Answer:_

<details> <summary>Answer</summary> After the memory allocation phase is complete. </details>

---

### 18. Can new Execution Contexts be created during program execution?

➡️ _Your Answer:_

<details> <summary>Answer</summary> Yes, whenever a function is invoked, a new Execution Context is created. </details>

---

### 19. What happens inside a new function Execution Context?

➡️ _Your Answer:_

<details> <summary>Answer</summary> The same two phases happen again: Memory creation and Code execution. </details>

---

### 20. Why is understanding Execution Context important?

➡️ _Your Answer:_

<details> <summary>Answer</summary> Because it helps in understanding how JavaScript runs code behind the scenes — vital for debugging and advanced learning. </details>

---

## Hoisting in JavaScript

```js
console.log(a); // undefined
var a = 10;
```

Function declaration hoisted with body:

```js
foo(); // "Hello"
function foo() {
  console.log('Hello');
}
```

Function expression not hoisted:

```js
bar(); // TypeError
var bar = function () {};
```

---

## Function vs Variable Invocation

- **Function invocation:** `foo()`
- **Variable invocation:** `console.log(x)`

---

## What Happens on an Empty JS File?

- Global Execution Context created
- `this === window` is true
- No output or error

---

## Window & This Keyword

- In global scope:
  - `this` refers to `window`
  - `var` declared variables attach to `window`

---

## Defined vs Undefined

- **Defined:** Assigned a value
- **Undefined:** Declared but no value assigned

```js
var a;
console.log(a); // undefined
```

---

## Scope, Scope Chain & Lexical Environment

- **Scope:** Access region for variables/functions
- **Scope Chain:** Nested chain of parent scopes
- **Lexical Environment:** Code + memory context

---

## var, let, const Differences

| Feature       | `var`             | `let`           | `const`         |
| ------------- | ----------------- | --------------- | --------------- |
| Scope         | Function scope    | Block scope     | Block scope     |
| Hoisting      | Yes (`undefined`) | Yes (TDZ error) | Yes (TDZ error) |
| Reassignment  | Yes               | Yes             | No              |
| Redeclaration | Yes               | No              | No              |

---

## Errors: SyntaxError vs ReferenceError vs TypeError

- **SyntaxError:** Bad syntax
  ```js
  var a = ; // SyntaxError
  ```
- **ReferenceError:** Accessing undeclared variable
  ```js
  console.log(x); // ReferenceError
  ```
- **TypeError:** Wrong operation on value type
  ```js
  null(); // TypeError: null is not a function
  ```

---

## Block, Compound Statement, Scope & Shadowing

- **Block:** `{ ... }`
- **Compound Statement:** Multiple statements in a block
- **Scope:** Context for variable access
- **Shadowing:** Inner variable with same name as outer

```js
let x = 1;
{
  let x = 2; // shadows outer x
}
```

---

## Closures in JavaScript

- A closure is when a function "remembers" its lexical scope even when run outside it.

```js
function outer() {
  let count = 0;
  return function inner() {
    count++;
    console.log(count);
  };
}

const counter = outer();
counter(); // 1
counter(); // 2
```
