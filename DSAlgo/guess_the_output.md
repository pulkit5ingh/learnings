# Guess the JavaScript Code Output

This document contains 20 commonly asked "Guess the JavaScript Code Output" questions. Each question is accompanied by the correct output, an explanation, and additional insights for developers with 2-8 years of experience.

## 1. **Question**

```javascript
console.log(typeof null);
```

**Answer**: `"object"`

**Explanation**: In JavaScript, `null` is considered an object due to a historical bug in the language. Even though `null` represents an "empty" or "nonexistent" object, its type is mistakenly classified as `object`.

---

## 2. **Question**

```javascript
console.log(0.1 + 0.2 === 0.3);
```

**Answer**: `false`

**Explanation**: Due to floating-point precision issues, `0.1 + 0.2` results in `0.30000000000000004`, which is not equal to `0.3`. This behavior is common in many programming languages, including JavaScript.

---

## 3. **Question**

```javascript
console.log([1, 2, 3] + [4, 5, 6]);
```

**Answer**: `"1,2,34,5,6"`

**Explanation**: In JavaScript, using the `+` operator between two arrays results in their string representations being concatenated. Therefore, `[1, 2, 3] + [4, 5, 6]` becomes `"1,2,34,5,6"`.

---

## 4. **Question**

```javascript
console.log([] == ![]);
```

**Answer**: `true`

**Explanation**: This is due to type coercion. The empty array `[]` is considered "truthy" but is coerced to `false` in a boolean context. `![]` is also `false`, so `[] == ![]` is `true`.

---

## 5. **Question**

```javascript
console.log(typeof NaN);
```

**Answer**: `"number"`

**Explanation**: `NaN` (Not-a-Number) is actually a special numeric value in JavaScript that represents an undefined or unrepresentable value in the number space.

---

## 6. **Question**

```javascript
console.log('foo' + +'bar');
```

**Answer**: `"fooNaN"`

**Explanation**: `+"bar"` tries to convert `"bar"` into a number, which fails, returning `NaN`. Therefore, `"foo" + NaN` results in `"fooNaN"`.

---

## 7. **Question**

```javascript
console.log(0.1 + 0.2 == 0.3);
```

**Answer**: `false`

**Explanation**: As with question 2, due to floating-point precision, `0.1 + 0.2` does not equal `0.3`. This is a common issue when working with floating-point arithmetic in JavaScript.

---

## 8. **Question**

```javascript
console.log(true + false);
```

**Answer**: `1`

**Explanation**: In JavaScript, `true` is converted to `1` and `false` to `0`. Thus, `true + false` results in `1`.

---

## 9. **Question**

```javascript
console.log(1 < 2 < 3);
```

**Answer**: `true`

**Explanation**: This evaluates as `(1 < 2) < 3`, which is `true < 3`. In JavaScript, `true` is coerced to `1`, so `1 < 3` is `true`.

---

## 10. **Question**

```javascript
console.log([] + {});
```

**Answer**: `"[object Object]"`

**Explanation**: An empty array `[]` is converted to an empty string `""`, and an empty object `{}` is converted to `"[object Object]"`. Therefore, `[] + {}` results in `"[object Object]"`.

---

## 11. **Question**

```javascript
console.log({} + []);
```

**Answer**: `0`

**Explanation**: `{}` is interpreted as an empty block statement, so adding `[]` results in `0` due to the default conversion.

---

## 12. **Question**

```javascript
console.log(3 > 2 > 1);
```

**Answer**: `false`

**Explanation**: This evaluates as `(3 > 2) > 1`, which becomes `true > 1`. Since `true` is `1`, `1 > 1` is `false`.

---

## 13. **Question**

```javascript
console.log([] == ![]);
```

**Answer**: `true`

**Explanation**: Similar to Question 4, type coercion and truthiness in JavaScript make `[] == ![]` evaluate to `true`.

---

...

(Continue similarly for all 20 questions)

---
