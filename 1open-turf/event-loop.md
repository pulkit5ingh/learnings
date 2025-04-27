# Event Loop in JavaScript

The event loop is a core concept in JavaScript and Node.js for handling asynchronous operations. It allows JavaScript to manage multiple tasks without blocking, even though JavaScript is single-threaded.

## Event Loop Overview

The event loop handles the execution of asynchronous code in JavaScript. Since JavaScript operates in a single-threaded environment, the event loop manages asynchronous operations like I/O, timers, and network requests without blocking the main thread.

### Event Loop Phases

The event loop consists of several phases that each handle specific types of operations:

1. **Timers**: Executes callbacks from `setTimeout` and `setInterval`.
2. **Pending Callbacks**: Executes I/O callbacks deferred to the next loop cycle.
3. **Idle, Prepare**: Internal operations (not commonly exposed).
4. **Poll**: Fetches new I/O events and executes their callbacks.
5. **Check**: Executes `setImmediate` callbacks.
6. **Close Callbacks**: Executes close callbacks, like `socket.on('close')`.

The event loop cycles through these phases, processing each phase until all callbacks are executed.

## Code Example

Here is an example illustrating the event loop's behavior:

```javascript
console.log('Start');

// Schedule a callback using setTimeout
setTimeout(() => {
  console.log('Timeout callback');
}, 0);

// Schedule a callback using setImmediate
setImmediate(() => {
  console.log('Immediate callback');
});

// A promise callback
Promise.resolve().then(() => {
  console.log('Promise callback');
});

console.log('End');
```

### Execution Order Explanation

1. **Synchronous Code**: `console.log('Start')` and `console.log('End')` are synchronous and execute first.
2. **Microtasks Queue (Promises)**: The `Promise.resolve().then()` callback executes after the synchronous code because promises run in the microtask queue, which has higher priority than other asynchronous tasks.
3. **Timers Phase**: `setTimeout` with a delay of 0 is executed during the **Timers phase**.
4. **Check Phase (setImmediate)**: The `setImmediate` callback executes in the **Check phase** after the timers.

**Expected Output**:

```
Start
End
Promise callback
Immediate callback
Timeout callback
```

### Why This Order?

1. **Start** and **End** log first as they are synchronous.
2. **Promise callback** executes next due to its priority in the microtask queue.
3. **Immediate callback** follows as it’s in the **Check phase**.
4. **Timeout callback** executes last in the **Timers phase**.

## Benefits of the Event Loop

The event loop enables JavaScript to handle asynchronous tasks efficiently, ensuring non-blocking execution for I/O-bound tasks. It leverages asynchronous capabilities even in a single-threaded environment, making JavaScript suitable for handling numerous I/O operations.

## Summary

- The event loop is essential for handling asynchronous operations in JavaScript.
- It consists of multiple phases, including Timers, Pending Callbacks, Poll, and Check.
- `Promise` callbacks have higher priority and are processed in the microtask queue.
- The event loop’s non-blocking nature makes JavaScript and Node.js highly efficient for I/O-intensive applications.
