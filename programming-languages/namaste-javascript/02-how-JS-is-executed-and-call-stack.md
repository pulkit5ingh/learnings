# Understanding Execution Context in JavaScript

## What happens when you run a JavaScript program?

When you run a JavaScript program, an **Execution Context** is created. JavaScript isn't possible without this beautiful Execution Context.

### JavaScript Program Example:

```javascript
var n = 2;
function square(num) {
  var ans = num * num;
  return ans;
}
var square2 = square(n);
var square4 = square(4);
```

### Creation of Execution Context:

The **Execution Context** has two components:

- **Memory Component** (also known as Variable Environment)
- **Code Component** (also known as Thread of Execution)

This Execution Context is created in two phases:

### 1. Memory Creation Phase (or Creation Phase)

- JavaScript scans the program and allocates memory to variables and functions.
- Variables like `n`, `square2`, and `square4` are assigned `undefined`.
- Functions like `square` are fully stored (their entire code) in memory.

### 2. Code Execution Phase

- JavaScript runs through the code line-by-line and executes it.
- Assigns actual values to variables and executes functions.

---

### Execution Steps:

1. Memory Allocation:

   - `n` → `undefined`
   - `square` → function code
   - `square2` → `undefined`
   - `square4` → `undefined`

2. Execution:
   - `n = 2`
   - `square2 = square(n)` → **Function Invocation**:
     - New Execution Context created.
     - Memory phase: `num` and `ans` assigned `undefined`.
     - Code phase: `num = 2`, `ans = num * num = 4`, return `4`.
     - Context destroyed after return.
   - `square4 = square(4)` → **Function Invocation**:
     - New Execution Context created.
     - Memory phase: `num` and `ans` assigned `undefined`.
     - Code phase: `num = 4`, `ans = num * num = 16`, return `16`.
     - Context destroyed after return.

---

### Important Points:

- **Function Invocation** creates a new **Execution Context**.
- After function execution, the **Execution Context** is **deleted**.
- Execution Contexts are managed in a **stack** (known as **Call Stack**).

---

> Don't you think that all this is too much to manage for the JavaScript engine?
> Suppose if there was a function invocation inside a function... 🤔
> (Stay tuned for more on this!)

---
