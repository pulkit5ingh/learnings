# JavaScript Interview Questions for 2-5 Years Experience

## 1. Explain the concept of closures in JavaScript.

A closure is when a function retains access to its lexical scope, even when the function is executed outside of its original scope. This allows inner functions to access variables defined in outer functions.

## 2. What is the difference between `let`, `const`, and `var`?

`var` is function-scoped and allows re-declaration, whereas `let` and `const` are block-scoped and don't allow re-declaration. `const` defines constants that cannot be reassigned, while `let` allows reassignment.

## 3. What are JavaScript promises?

Promises represent the eventual completion (or failure) of an asynchronous operation, providing a cleaner way to handle async logic than callbacks.

## 4. Explain async/await in JavaScript.

`async` functions return a promise and `await` is used to wait for the promise to resolve, making async code look synchronous and easier to read.

## 5. What is hoisting in JavaScript?

Hoisting is JavaScript's default behavior of moving declarations to the top of the scope before execution, allowing `var` and function declarations to be used before they are declared.

## 6. What are higher-order functions?

Higher-order functions take other functions as arguments or return functions as their result. Examples include `map`, `filter`, and `reduce`.

## 7. Explain the difference between `==` and `===` in JavaScript.

`==` performs type conversion before comparison (loose equality), while `===` does not (strict equality).

## 8. What is event bubbling and capturing?

Event bubbling is when an event propagates from the target element up to the DOM root, while capturing goes from the root down to the target.

## 9. Explain `this` keyword in JavaScript.

`this` refers to the object that the function is a property of. Its value depends on the context in which the function is called.

## 10. What are arrow functions, and how are they different from regular functions?

Arrow functions are a concise syntax for function expressions that don’t have their own `this`, `arguments`, or `prototype`, inheriting `this` from their enclosing scope.

## 11. What is the purpose of the `bind` method?

`bind` creates a new function with `this` keyword and, optionally, initial arguments set to the specified values.

## 12. Explain the concept of prototypal inheritance.

JavaScript objects can inherit properties from other objects. Each object has a prototype from which it can inherit properties.

## 13. How does the `call` method work in JavaScript?

`call` invokes a function with a specified `this` value and arguments provided individually.

## 14. How does the `apply` method work in JavaScript?

`apply` is similar to `call`, but it takes arguments as an array.

## 15. What is the purpose of the `new` keyword in JavaScript?

`new` is used to create an instance of an object with a constructor function, setting up inheritance and initializing properties.

## 16. What is the difference between `slice` and `splice`?

`slice` returns a shallow copy of an array without modifying the original, while `splice` modifies the array by adding or removing elements.

## 17. Explain the `map` function in JavaScript.

`map` creates a new array by applying a function to each element of an array.

## 18. What is the purpose of the `reduce` function?

`reduce` executes a reducer function on each element of an array, resulting in a single output value.

## 19. Explain the concept of `NaN` in JavaScript.

`NaN` stands for "Not-a-Number" and is returned when a mathematical operation cannot produce a valid number.

## 20. What is the use of `typeof` operator?

`typeof` returns a string indicating the data type of a variable.

## 21. What are template literals?

Template literals allow multi-line strings and expression interpolation with backticks (`` ` ``) instead of quotes.

## 22. Explain the difference between `null` and `undefined`.

`null` is an assigned value representing "no value," while `undefined` means a variable has been declared but has not yet been assigned a value.

## 23. How does destructuring work in JavaScript?

Destructuring allows unpacking values from arrays or properties from objects into distinct variables.

## 24. What is JSON and how do you parse it?

JSON (JavaScript Object Notation) is a format for storing and transporting data. Use `JSON.parse` to parse a JSON string into an object and `JSON.stringify` to convert an object into a JSON string.

## 25. Explain how `setTimeout` and `setInterval` work.

`setTimeout` delays the execution of a function by a specified time, while `setInterval` repeatedly calls a function at specified intervals.

## 26. What are modules in JavaScript?

Modules allow encapsulation of code by exporting and importing functionality, using `export` and `import` statements.

## 27. What is event delegation?

Event delegation allows you to add a single event listener to a parent element that handles events from its children, using the event's `target` property.

## 28. Explain how `async` and `defer` work for script loading.

`async` loads the script asynchronously and executes it as soon as it’s ready, while `defer` loads the script asynchronously but executes it only after the HTML parsing is complete.

## 29. What is a callback function?

A callback is a function passed into another function as an argument and is executed after the completion of that function.

## 30. What is the DOM?

The DOM (Document Object Model) represents the HTML document structure as a tree, allowing JavaScript to manipulate and access elements.

## 31. What are IIFEs (Immediately Invoked Function Expressions)?

IIFEs are functions that execute as soon as they are defined, commonly used to create a private scope.

## 32. What is the difference between synchronous and asynchronous code?

Synchronous code executes in sequence, blocking the next operation until the current one completes, whereas asynchronous code doesn’t wait and allows other operations to continue.

## 33. What is functional programming?

Functional programming is a paradigm that emphasizes immutability and pure functions, avoiding shared state and side effects.

## 34. What is an object literal?

An object literal is a syntax to create an object by defining its properties within curly braces.

## 35. What is variable shadowing?

Variable shadowing occurs when a variable declared within a certain scope has the same name as a variable in an outer scope.

## 36. What is memoization?

Memoization is an optimization technique where the results of expensive function calls are cached, improving performance by avoiding repeated calculations.

## 37. How does `fetch` work in JavaScript?

`fetch` is a modern way to make HTTP requests and returns a promise, allowing async requests to APIs.

## 38. What is the purpose of `Object.assign()`?

`Object.assign()` is used to copy properties from one or more source objects to a target object.

## 39. How do you clone an object in JavaScript?

Use `Object.assign()` or the spread operator `{...obj}` to create a shallow clone of an object.

## 40. What are symbols in JavaScript?

Symbols are unique and immutable primitive values used as unique object property keys.

## 41. How do `for...of` and `for...in` differ?

`for...of` iterates over iterable objects like arrays, while `for...in` iterates over the enumerable properties of an object.

## 42. What is a generator function?

A generator function allows pausing and resuming function execution using the `yield` keyword.

## 43. Explain throttling and debouncing.

Throttling limits the execution of a function to once per time interval, while debouncing delays the function execution until a specified time has passed since the last call.

## 44. How do you convert a NodeList to an array?

Use `Array.from(nodeList)` or the spread operator `[...nodeList]`.

## 45. What is the Event Loop?

The Event Loop is JavaScript's mechanism to handle asynchronous callbacks by managing the call stack and callback queue.

## 46. Explain the concept of "strict mode" in JavaScript.

"Strict mode" enables stricter parsing and error handling in JavaScript, helping to catch potential errors early.

## 47. What are JavaScript decorators?

Decorators are a proposal for applying annotations and a meta-programming syntax for class declarations and properties.

## 48. What is the WeakMap in JavaScript?

`WeakMap` is a collection of key-value pairs where the keys are objects and the values can be any type. It doesn’t prevent garbage collection if the key object is no longer referenced elsewhere, helping manage memory more efficiently.

## 49. What is the difference between a shallow copy and a deep copy?

A shallow copy only copies the first level of an object, while a deep copy recursively copies all levels. Shallow copies may cause unexpected mutations in nested objects, while deep copies avoid this by duplicating every layer.

## 50. How do you deep clone an object in JavaScript?

You can deep clone an object using structured cloning (`structuredClone()`), `JSON.parse(JSON.stringify(obj))` for simple data, or external libraries like Lodash’s `_.cloneDeep()`.
