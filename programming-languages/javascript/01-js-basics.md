https://chatgpt.com/share/e3432bfa-8e65-4141-8d95-c408c2e89a0b

### statements

```javascript
ex:-

    let x, y, z; // Statement 1
    x = 5; // Statement 2
    y = 6; // Statement 3
    z = x + y; // Statement 4

```

### semicolons

- semicolons separate JavaScript statements.
- add a semicolon at the end of each executable statement:

```javascript
ex:-

    let a, b, c; // Declare 3 variables
    a = 5; // Assign the value 5 to a
    b = 6; // Assign the value 6 to b
    c = a + b; // Assign the sum of a and b to c

```

### syntax

- JavaScript syntax is the set of rules, how JavaScript programs are constructed:-

- How to create variables:

```javascript
ex:-

    var x;
    let y;

```

- How to use variables:

```javascript
ex:-

    x = 5;
    y = 6;
    let z = x + y;

```

### javascript variables

1. Variables:
   JavaScript variables are containers for storing data values. You can declare a variable using var, let, or const.

   var: Function-scoped and can be re-assigned.
   let: Block-scoped and can be re-assigned.
   const: Block-scoped and cannot be re-assigned.

```javascript
ex:-

var name = 'John';
let age = 30;
const country = 'USA';

```

### javascript data types

- JavaScript has several data types categorized into two types: primitive and reference.

- Primitive Types:

  1. String: Textual data. Example: 'Hello, World!'
  2. Number: Numeric data. Example: 42
  3. Boolean: Logical data. Example: true or false
  4. Null: Represents intentional absence of value. Example: null
  5. Undefined: Represents an uninitialized variable. Example: undefined
  6. Symbol: Unique and immutable value used as the key of an object property.
  7. BigInt: For arbitrarily large integers.

```javascript
    ex:-

    let message = 'Hello, World!'; // String
    let count = 42; // Number
    let isActive = true; // Boolean
    let data = null; // Null
    let value; // Undefined
    let uniqueID = Symbol('id'); // Symbol
    let largeNumber = 9007199254740991n; // BigInt

```

- Non-Primitive Types:

  1. Object: Collection of key-value pairs.
  2. Array: Ordered collection of values.
  3. Function: Block of code designed to perform a particular task.

```javascript
    ex:-

    let person = { name: 'Alice', age: 25 }; // Object
    let numbers = [1, 2, 3, 4, 5]; // Array
    function greet(name) { return 'Hello, ' + name; } // Function

```

### javascript operators

1. Arithmetic Operators: Used to perform arithmetic on numbers.

```javascript
    ex:-

    let a = 5;
    let b = 2;

    console.log(a + b); // Addition: 7
    console.log(a - b); // Subtraction: 3
    console.log(a * b); // Multiplication: 10
    console.log(a / b); // Division: 2.5
    console.log(a % b); // Modulus: 1
    console.log(a ** b); // Exponentiation: 25

```

2. Assignment Operators: Used to assign values to variables.

```javascript
    ex:-

    let x = 10;
    x += 5; // Equivalent to x = x + 5
    console.log(x); // 15

    x *= 2; // Equivalent to x = x * 2
    console.log(x); // 30

```

3. Comparison Operators: Used to compare two values.

```javascript
ex: -console.log(5 == '5'); // Equal to: true
console.log(5 === '5'); // Strict equal to: false
console.log(5 != '5'); // Not equal to: false
console.log(5 !== '5'); // Strict not equal to: true
console.log(5 > 2); // Greater than: true
console.log(5 < 2); // Less than: false
```

4. Logical Operators: Used to combine logical statements.

```javascript
    ex:-

    let a = true;
    let b = false;

    console.log(a && b); // Logical AND: false
    console.log(a || b); // Logical OR: true
    console.log(!a); // Logical NOT: false

```

### javascript control structures

1. Conditional Statements: Used to perform different actions based on different conditions.

```javascript
    ex:-

    let num = 10;

    if (num > 0) {
    console.log('Positive number');
    } else if (num < 0) {
    console.log('Negative number');
    } else {
    console.log('Zero');
    }

```

2. Switch Statement: Used to perform different actions based on different conditions.

```javascript
    ex:-

    let day = 'Monday';

    switch (day) {
    case 'Monday':
        console.log('Start of the work week');
        break;
    case 'Friday':
        console.log('End of the work week');
        break;
    default:
        console.log('Middle of the week');
    }

```

3. Loops: Used to repeat a block of code a number of times.

- 1. For Loop:

```javascript
    ex:-

    for (let i = 0; i < 5; i++) {
        console.log(i);
    }

```

- 2. While Loop:

```javascript
    ex:-

    for (let i = 0; i < 5; i++) {
        console.log(i);
    }

```

- 3. Do-While Loop:

```javascript
    ex:-

    let i = 0;
    do {
        console.log(i);
        i++;
    } while (i < 5);

```

- 4. Iteration Methods

```javascript
    ex:-

    let numbers = [1, 2, 3, 4, 5];

    numbers.forEach((num) => {
        console.log(num);
    });

    let squares = numbers.map((num) => {
        return num * num;
    });
    console.log(squares); // [1, 4, 9, 16, 25]

    let evens = numbers.filter((num) => {
        return num % 2 === 0;
    });
    console.log(evens); // [2, 4]

    let sum = numbers.reduce((total, num) => {
        return total + num;
    }, 0);
    console.log(sum); // 15

```

### javascript functions

1. Function declaration

```javascript
ex: -function add(a, b) {
  return a + b;
};

console.log(add(2, 3)); // 5
```

2. Function Expression

```javascript
    ex:-

    const multiply = function(a, b) {
    return a * b;
    };

    console.log(multiply(2, 3)); // 6

```

3. Arrow Functions:

```javascript
    ex:-

    const subtract = (a, b) => a - b;

    console.log(subtract(5, 2)); // 3

```

4. Higher-Order Functions: Functions that operate on other functions.

```javascript
ex: -function filterArray(arr, callback) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    if (callback(arr[i])) {
      result.push(arr[i]);
    }
  }
  return result;
};

const numbers = [1, 2, 3, 4, 5];
const evenNumbers = filterArray(numbers, (num) => num % 2 === 0);

console.log(evenNumbers); // [2, 4]
```

5. Callback Functions: Functions passed as arguments to other functions.

```javascript
ex: -function greet(name, callback) {
  console.log('Hello, ' + name);
  callback();
};

function sayGoodbye() {
  console.log('Goodbye!');
}

greet('Alice', sayGoodbye);
// Output:
// Hello, Alice
// Goodbye!
```

6. Closures: Functions that remember the scope in which they were created.

```javascript
ex: -function makeCounter() {
  let count = 0;
  return function () {
    count++;
    return count;
  };
};

const counter = makeCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```
