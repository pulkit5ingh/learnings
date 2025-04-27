# JavaScript Data Types

JavaScript has various data types classified into **Primitive** and **Non-Primitive (Reference)** types. Here's an explanation of each with examples and differences:

---

## 1. Primitive Data Types

Primitive types are immutable, meaning their values cannot be changed after they are created. They are stored directly in memory.

| **Data Type** | **Example**                 | **Description**                                                                                           |
| ------------- | --------------------------- | --------------------------------------------------------------------------------------------------------- |
| **String**    | `"hello"`, `'world'`        | Represents a sequence of characters (text). Enclosed in single (`''`), double (`""`), or backticks (` `). |
| **Number**    | `42`, `3.14`, `NaN`         | Represents both integers and floating-point numbers. Includes `NaN` (Not-a-Number) and `Infinity`.        |
| **BigInt**    | `123n`, `9007199254740991n` | Used for numbers larger than `Number.MAX_SAFE_INTEGER` (2^53 - 1). Add `n` to create a BigInt.            |
| **Boolean**   | `true`, `false`             | Represents logical values, used in conditions to represent "yes/no" or "on/off".                          |
| **Undefined** | `undefined`                 | Indicates a variable has been declared but not assigned a value.                                          |
| **Null**      | `null`                      | Represents an intentional absence of value. Often used to reset or clear a variable.                      |
| **Symbol**    | `Symbol('id')`              | Represents a unique identifier, mainly used as property keys for objects to avoid collisions.             |

---

## 2. Non-Primitive (Reference) Data Types

Non-primitive types are mutable and stored by reference. They are used to store collections of data or more complex entities.

| **Data Type** | **Example**               | **Description**                                                                                  |
| ------------- | ------------------------- | ------------------------------------------------------------------------------------------------ |
| **Object**    | `{name: 'John', age: 30}` | A collection of key-value pairs. Keys are strings (or Symbols), and values can be any type.      |
| **Array**     | `[1, 2, 3, "apple"]`      | A list-like ordered collection of values. Access elements via index (e.g., `array[0]`).          |
| **Function**  | `function greet() {}`     | A reusable block of code that can be executed. Functions are first-class citizens in JavaScript. |
| **Date**      | `new Date()`              | Represents date and time. Provides methods for manipulating dates and times.                     |
| **RegExp**    | `/pattern/`               | Represents regular expressions used for pattern matching and string searching.                   |
| **Map**       | `new Map()`               | A collection of key-value pairs where keys can be of any type, unlike plain objects.             |
| **Set**       | `new Set([1, 2, 3])`      | A collection of unique values, preventing duplicates.                                            |
| **WeakMap**   | `new WeakMap()`           | Similar to `Map`, but keys are weakly held, meaning they can be garbage-collected.               |
| **WeakSet**   | `new WeakSet()`           | Similar to `Set`, but holds objects weakly to prevent memory leaks.                              |

---

## 3. Key Differences Between Primitive and Non-Primitive Types

| **Aspect**        | **Primitive Types**                    | **Non-Primitive Types**                          |
| ----------------- | -------------------------------------- | ------------------------------------------------ |
| **Mutability**    | Immutable: Values cannot be changed.   | Mutable: Values can be modified.                 |
| **Storage**       | Stored directly in memory.             | Stored as a reference to a memory location.      |
| **Copy Behavior** | Copied by value (creates a new value). | Copied by reference (points to the same object). |
| **Examples**      | `string`, `number`, `boolean`, etc.    | `object`, `array`, `function`, etc.              |

---

## 4. Special Cases

### **`typeof` Operator**

- **`typeof null`**: Returns `"object"` due to a historical bug in JavaScript, but `null` is not an object.
- **`typeof NaN`**: Returns `"number"`, even though it means "Not-a-Number."
- **`typeof function`**: Returns `"function"`, which is a subtype of `object`.

### **Dynamic Typing**

JavaScript allows variables to hold values of different types at runtime:

```javascript
let value = 10; // Number
value = 'Ten'; // String
value = true; // Boolean
```

---

## 5. Examples for Each Data Type

### **Primitive Types**

```javascript
let str = 'Hello, World'; // String
let num = 42; // Number
let big = 123456789012345678901n; // BigInt
let bool = true; // Boolean
let notDefined; // Undefined
let emptyValue = null; // Null
let uniqueId = Symbol('id'); // Symbol
```

### **Non-Primitive Types**

```javascript
let obj = { name: 'John', age: 25 }; // Object
let arr = [1, 2, 3, 4]; // Array
let greet = function () {
  // Function
  console.log('Hello');
};
let today = new Date(); // Date
let regex = /abc/; // RegExp
let map = new Map(); // Map
let set = new Set([1, 2, 3]); // Set
```

---

## 6. Summary of `typeof` Results

| **Expression**        | **Result**    |
| --------------------- | ------------- |
| `typeof "hello"`      | `"string"`    |
| `typeof 42`           | `"number"`    |
| `typeof 123n`         | `"bigint"`    |
| `typeof true`         | `"boolean"`   |
| `typeof undefined`    | `"undefined"` |
| `typeof null`         | `"object"`    |
| `typeof Symbol()`     | `"symbol"`    |
| `typeof {}`           | `"object"`    |
| `typeof []`           | `"object"`    |
| `typeof function(){}` | `"function"`  |

---

### Primitive type

- number

```javascript
concat(index);
filter(index);
find(index);
map(index);
push(index);
pop(index);
```

- string

```javascript
charAt(index);
includes(index);
replace(index);
slice(index);
toUpperCase(index);
split(index);
```

- bigint
- boolean
- null
- undefined
- symbol

### Non-Primitive type

- Object

```javascript
assign(index);
keys(index);
values(index);
entries(index);
```

### Javascript object

- Prototypal inheritance
- Object prototype
- Built-in object
