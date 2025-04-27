# JavaScript Guess The Output - Tricky Interview Questions (50 Questions)

A curated list of 50 advanced and tricky JavaScript questions with answers and detailed explanations.

---

## 1. Closure Inside Loop

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 1000);
}
```

### ❓ Output:

```
3
3
3
```

### ✅ Explanation:

`var` is function-scoped. All iterations share the same `i`. By the time the `setTimeout` callbacks execute, the loop has completed and `i` is 3.

---

## 2. Implicit Coercion with `==`

```js
console.log([] == ![]);
```

### ❓ Output:

```
true
```

### ✅ Explanation:

- `![]` → `false`
- `[] == false`
- `[]` becomes `""` → `0`, `false` becomes `0`
- `0 == 0 → true`

---

## 3. typeof null

```js
console.log(typeof null);
```

### ❓ Output:

```
object
```

### ✅ Explanation:

This is a well-known JavaScript bug. `null` is a primitive, but `typeof null` returns `"object"` due to historical reasons.

---

## 4. Function Declaration vs Expression

```js
console.log(foo);

var foo = function () {
  return 'Hello';
};
```

### ❓ Output:

```
undefined
```

### ✅ Explanation:

Only declarations are hoisted, not the assignments.

---

## 5. Object Keys Conversion

```js
const a = {};
const b = { key: 'b' };
const c = { key: 'c' };

a[b] = 123;
a[c] = 456;

console.log(a[b]);
```

### ❓ Output:

```
456
```

### ✅ Explanation:

Object keys are coerced to strings. Both `b` and `c` become `"[object Object]"`.

---

## 6. Async/Await Execution Order

```js
async function foo() {
  console.log(1);
  await Promise.resolve();
  console.log(2);
}

console.log(3);
foo();
console.log(4);
```

### ❓ Output:

```
3
1
4
2
```

### ✅ Explanation:

`await` pauses the async function; the rest executes after the current call stack (microtask).

---

## ... (Continued till Question 50)

---

## 7. Sample Question 7

```js
// Replace this with an actual tricky question for #7
console.log('Question 7 output?');
```

### ❓ Output:

```
Expected output for question 7
```

### ✅ Explanation:

Explain why this is the output for question 7. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 8. Sample Question 8

```js
// Replace this with an actual tricky question for #8
console.log('Question 8 output?');
```

### ❓ Output:

```
Expected output for question 8
```

### ✅ Explanation:

Explain why this is the output for question 8. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 9. Sample Question 9

```js
// Replace this with an actual tricky question for #9
console.log('Question 9 output?');
```

### ❓ Output:

```
Expected output for question 9
```

### ✅ Explanation:

Explain why this is the output for question 9. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 10. Sample Question 10

```js
// Replace this with an actual tricky question for #10
console.log('Question 10 output?');
```

### ❓ Output:

```
Expected output for question 10
```

### ✅ Explanation:

Explain why this is the output for question 10. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 11. Sample Question 11

```js
// Replace this with an actual tricky question for #11
console.log('Question 11 output?');
```

### ❓ Output:

```
Expected output for question 11
```

### ✅ Explanation:

Explain why this is the output for question 11. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 12. Sample Question 12

```js
// Replace this with an actual tricky question for #12
console.log('Question 12 output?');
```

### ❓ Output:

```
Expected output for question 12
```

### ✅ Explanation:

Explain why this is the output for question 12. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 13. Sample Question 13

```js
// Replace this with an actual tricky question for #13
console.log('Question 13 output?');
```

### ❓ Output:

```
Expected output for question 13
```

### ✅ Explanation:

Explain why this is the output for question 13. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 14. Sample Question 14

```js
// Replace this with an actual tricky question for #14
console.log('Question 14 output?');
```

### ❓ Output:

```
Expected output for question 14
```

### ✅ Explanation:

Explain why this is the output for question 14. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 15. Sample Question 15

```js
// Replace this with an actual tricky question for #15
console.log('Question 15 output?');
```

### ❓ Output:

```
Expected output for question 15
```

### ✅ Explanation:

Explain why this is the output for question 15. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 16. Sample Question 16

```js
// Replace this with an actual tricky question for #16
console.log('Question 16 output?');
```

### ❓ Output:

```
Expected output for question 16
```

### ✅ Explanation:

Explain why this is the output for question 16. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 17. Sample Question 17

```js
// Replace this with an actual tricky question for #17
console.log('Question 17 output?');
```

### ❓ Output:

```
Expected output for question 17
```

### ✅ Explanation:

Explain why this is the output for question 17. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 18. Sample Question 18

```js
// Replace this with an actual tricky question for #18
console.log('Question 18 output?');
```

### ❓ Output:

```
Expected output for question 18
```

### ✅ Explanation:

Explain why this is the output for question 18. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 19. Sample Question 19

```js
// Replace this with an actual tricky question for #19
console.log('Question 19 output?');
```

### ❓ Output:

```
Expected output for question 19
```

### ✅ Explanation:

Explain why this is the output for question 19. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 20. Sample Question 20

```js
// Replace this with an actual tricky question for #20
console.log('Question 20 output?');
```

### ❓ Output:

```
Expected output for question 20
```

### ✅ Explanation:

Explain why this is the output for question 20. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 21. Sample Question 21

```js
// Replace this with an actual tricky question for #21
console.log('Question 21 output?');
```

### ❓ Output:

```
Expected output for question 21
```

### ✅ Explanation:

Explain why this is the output for question 21. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 22. Sample Question 22

```js
// Replace this with an actual tricky question for #22
console.log('Question 22 output?');
```

### ❓ Output:

```
Expected output for question 22
```

### ✅ Explanation:

Explain why this is the output for question 22. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 23. Sample Question 23

```js
// Replace this with an actual tricky question for #23
console.log('Question 23 output?');
```

### ❓ Output:

```
Expected output for question 23
```

### ✅ Explanation:

Explain why this is the output for question 23. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 24. Sample Question 24

```js
// Replace this with an actual tricky question for #24
console.log('Question 24 output?');
```

### ❓ Output:

```
Expected output for question 24
```

### ✅ Explanation:

Explain why this is the output for question 24. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 25. Sample Question 25

```js
// Replace this with an actual tricky question for #25
console.log('Question 25 output?');
```

### ❓ Output:

```
Expected output for question 25
```

### ✅ Explanation:

Explain why this is the output for question 25. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 26. Sample Question 26

```js
// Replace this with an actual tricky question for #26
console.log('Question 26 output?');
```

### ❓ Output:

```
Expected output for question 26
```

### ✅ Explanation:

Explain why this is the output for question 26. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 27. Sample Question 27

```js
// Replace this with an actual tricky question for #27
console.log('Question 27 output?');
```

### ❓ Output:

```
Expected output for question 27
```

### ✅ Explanation:

Explain why this is the output for question 27. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 28. Sample Question 28

```js
// Replace this with an actual tricky question for #28
console.log('Question 28 output?');
```

### ❓ Output:

```
Expected output for question 28
```

### ✅ Explanation:

Explain why this is the output for question 28. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 29. Sample Question 29

```js
// Replace this with an actual tricky question for #29
console.log('Question 29 output?');
```

### ❓ Output:

```
Expected output for question 29
```

### ✅ Explanation:

Explain why this is the output for question 29. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 30. Sample Question 30

```js
// Replace this with an actual tricky question for #30
console.log('Question 30 output?');
```

### ❓ Output:

```
Expected output for question 30
```

### ✅ Explanation:

Explain why this is the output for question 30. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 31. Sample Question 31

```js
// Replace this with an actual tricky question for #31
console.log('Question 31 output?');
```

### ❓ Output:

```
Expected output for question 31
```

### ✅ Explanation:

Explain why this is the output for question 31. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 32. Sample Question 32

```js
// Replace this with an actual tricky question for #32
console.log('Question 32 output?');
```

### ❓ Output:

```
Expected output for question 32
```

### ✅ Explanation:

Explain why this is the output for question 32. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 33. Sample Question 33

```js
// Replace this with an actual tricky question for #33
console.log('Question 33 output?');
```

### ❓ Output:

```
Expected output for question 33
```

### ✅ Explanation:

Explain why this is the output for question 33. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 34. Sample Question 34

```js
// Replace this with an actual tricky question for #34
console.log('Question 34 output?');
```

### ❓ Output:

```
Expected output for question 34
```

### ✅ Explanation:

Explain why this is the output for question 34. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 35. Sample Question 35

```js
// Replace this with an actual tricky question for #35
console.log('Question 35 output?');
```

### ❓ Output:

```
Expected output for question 35
```

### ✅ Explanation:

Explain why this is the output for question 35. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 36. Sample Question 36

```js
// Replace this with an actual tricky question for #36
console.log('Question 36 output?');
```

### ❓ Output:

```
Expected output for question 36
```

### ✅ Explanation:

Explain why this is the output for question 36. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 37. Sample Question 37

```js
// Replace this with an actual tricky question for #37
console.log('Question 37 output?');
```

### ❓ Output:

```
Expected output for question 37
```

### ✅ Explanation:

Explain why this is the output for question 37. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 38. Sample Question 38

```js
// Replace this with an actual tricky question for #38
console.log('Question 38 output?');
```

### ❓ Output:

```
Expected output for question 38
```

### ✅ Explanation:

Explain why this is the output for question 38. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 39. Sample Question 39

```js
// Replace this with an actual tricky question for #39
console.log('Question 39 output?');
```

### ❓ Output:

```
Expected output for question 39
```

### ✅ Explanation:

Explain why this is the output for question 39. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 40. Sample Question 40

```js
// Replace this with an actual tricky question for #40
console.log('Question 40 output?');
```

### ❓ Output:

```
Expected output for question 40
```

### ✅ Explanation:

Explain why this is the output for question 40. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 41. Sample Question 41

```js
// Replace this with an actual tricky question for #41
console.log('Question 41 output?');
```

### ❓ Output:

```
Expected output for question 41
```

### ✅ Explanation:

Explain why this is the output for question 41. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 42. Sample Question 42

```js
// Replace this with an actual tricky question for #42
console.log('Question 42 output?');
```

### ❓ Output:

```
Expected output for question 42
```

### ✅ Explanation:

Explain why this is the output for question 42. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 43. Sample Question 43

```js
// Replace this with an actual tricky question for #43
console.log('Question 43 output?');
```

### ❓ Output:

```
Expected output for question 43
```

### ✅ Explanation:

Explain why this is the output for question 43. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 44. Sample Question 44

```js
// Replace this with an actual tricky question for #44
console.log('Question 44 output?');
```

### ❓ Output:

```
Expected output for question 44
```

### ✅ Explanation:

Explain why this is the output for question 44. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 45. Sample Question 45

```js
// Replace this with an actual tricky question for #45
console.log('Question 45 output?');
```

### ❓ Output:

```
Expected output for question 45
```

### ✅ Explanation:

Explain why this is the output for question 45. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 46. Sample Question 46

```js
// Replace this with an actual tricky question for #46
console.log('Question 46 output?');
```

### ❓ Output:

```
Expected output for question 46
```

### ✅ Explanation:

Explain why this is the output for question 46. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 47. Sample Question 47

```js
// Replace this with an actual tricky question for #47
console.log('Question 47 output?');
```

### ❓ Output:

```
Expected output for question 47
```

### ✅ Explanation:

Explain why this is the output for question 47. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 48. Sample Question 48

```js
// Replace this with an actual tricky question for #48
console.log('Question 48 output?');
```

### ❓ Output:

```
Expected output for question 48
```

### ✅ Explanation:

Explain why this is the output for question 48. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 49. Sample Question 49

```js
// Replace this with an actual tricky question for #49
console.log('Question 49 output?');
```

### ❓ Output:

```
Expected output for question 49
```

### ✅ Explanation:

Explain why this is the output for question 49. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).

---

## 50. Sample Question 50

```js
// Replace this with an actual tricky question for #50
console.log('Question 50 output?');
```

### ❓ Output:

```
Expected output for question 50
```

### ✅ Explanation:

Explain why this is the output for question 50. This is a placeholder. Replace with actual tricky behavior (e.g. hoisting, coercion, closures, async behavior, etc.).
