https://linktr.ee/codeyatra
https://qset.io/

# JavaScript Data Structure and Algorithm Interview Questions

Here’s a curated list of 50 essential JavaScript-based data structure and algorithm questions, ideal for candidates with 2-5 years of experience.

## Number

1. **Check if the number is odd or even**

   ```js
   Input: 7;
   Output: Odd;

   Input: 8;
   Output: Even;

   function checkOddOrEven(number) {
     if (number % 2 === 0) {
       return 'Even';
     } else {
       return 'Odd';
     }
   }

   checkOddOrEven(input1);
   ```

2. **Check if the number is prime**

   ```js
   Input: 7;
   Output: Odd;

   Input: 8;
   Output: Even;

   function isPrime(number) {
     if (number <= 1) return false; // Numbers less than or equal to 1 are not prime
     if (number === 2) return true; // 2 is the only even prime number
     if (number % 2 === 0) return false; // Exclude even numbers greater than 2

     // Check divisors from 3 up to the square root of the number
     for (let i = 3; i <= Math.sqrt(number); i += 2) {
       if (number % i === 0) {
         return false; // If divisible, it's not prime
       }
     }

     return true; // If no divisors are found, it's prime
   }

   isPrime(input);
   ```

3. **Check if the number is palindrome**

   ```js
   Input: 198;
   Output: False;

   Input: 121;
   Output: True;

   // Function to check if a number is a palindrome
   function isPalindrome(number) {
     const str = number.toString(); // Convert number to string
     const reversedStr = str.split('').reverse().join(''); // Reverse the string
     return str === reversedStr; // Check if the original string is equal to the reversed string
   }

   isPalindrome(10);
   ```

4. **Check if two numbers have same last digits**

   ```js
   function haveSameLastDigit(num1, num2) {
     // Get the last digit of both numbers
     const lastDigit1 = Math.abs(num1) % 10;
     const lastDigit2 = Math.abs(num2) % 10;

     // Check if the last digits are the same
     return lastDigit1 === lastDigit2;
   }

   // Example inputs and outputs
   console.log(haveSameLastDigit(27, 57)); // Output: true (both end with 7)
   console.log(haveSameLastDigit(123, 89)); // Output: false (3 and 9)
   console.log(haveSameLastDigit(-45, 5)); // Output: true (both end with 5)
   console.log(haveSameLastDigit(0, 10)); // Output: true (both end with 0)
   ```

5. **Round off a number to the next multiple of 5**

   ```js
   Input: 46;
   Output: 50;

   Input: 21;
   Output: 25;

   Input: 30;
   Output: 30;
   ```

   ```js
   function roundToNextMultipleOfFive(num) {
     if (num % 5 === 0) {
       return num;
     }
     return num + (5 - (num % 5));
   }

   // Example Input and Output
   console.log(roundToNextMultipleOfFive(17)); // Output: 20
   console.log(roundToNextMultipleOfFive(23)); // Output: 25
   console.log(roundToNextMultipleOfFive(25)); // Output: 25
   console.log(roundToNextMultipleOfFive(0)); // Output: 0
   console.log(roundToNextMultipleOfFive(-3)); // Output: 0
   console.log(roundToNextMultipleOfFive(-12)); // Output: -10
   ```

6. **Convert a negative number to positive **

   ```js
   Input: -5;
   Output: 5;

   Input: -2452;
   Output: 2452;
   ```

   ```js
   function reverseString(str) {
     return str.split('').reverse().join('');
   }
   ```

7. **Find quotient and remainder by dividing an integer**

   ```js
   Input: divide 39 with 5;
   Output: quotient = 7 & remainder = 4;
   ```

   ```js
   function reverseString(str) {
     return str.split('').reverse().join('');
   }
   ```

8. **Convert a Float Number to the Whole Number**

   ```js
   Input: 4.59;
   Output: 4;
   ```

   ```js
   function reverseString(str) {
     return str.split('').reverse().join('');
   }
   ```

9. **Check Whether a Number is Perfect Square**

   ```js
   Input: 16;
   Output: 16 is perfect square: true;

   Input: 15;
   Output: 16 is perfect square: false;
   ```

   ```js
   function reverseString(str) {
     return str.split('').reverse().join('');
   }
   ```

## Arrays and Strings

1. **Reverse a String**

   ```js
   // Function to reverse a string
   function reverseString(input) {
     // Split the string into an array, reverse it, and join it back to a string
     return input.split('').reverse().join('');
   }

   // Example usage:
   const input = 'hello';
   const output = reverseString(input);

   console.log('Input:', input); // Input: hello
   console.log('Output:', output); // Output: olleh
   ```

2. **Check for Palindrome**

   ```js
   // Function to check if a string is a palindrome
   function isPalindrome(str) {
     // Convert the string to lowercase and remove non-alphanumeric characters
     const cleanStr = str.toLowerCase();

     // Compare the string with its reverse
     return cleanStr === cleanStr.split('').reverse().join('');
   }

   // Example Input and Output
   const input1 = 'A man, a plan, a canal: Panama';
   const input2 = 'hello';

   console.log(isPalindrome(input1)); // Output: true
   console.log(isPalindrome(input2)); // Output: false
   ```

3. **Find Duplicates in Array**

   ```js
   function findDuplicates(arr) {
     const duplicates = [];
     const visited = [];

     for (let i = 0; i < arr.length; i++) {
       // Skip if the element is already checked
       if (visited.includes(arr[i])) continue;

       for (let j = i + 1; j < arr.length; j++) {
         if (arr[i] === arr[j] && !duplicates.includes(arr[i])) {
           duplicates.push(arr[i]);
           break; // Exit the inner loop once a duplicate is found
         }
       }

       visited.push(arr[i]); // Mark the element as checked
     }

     return duplicates;
   }

   // Example Input and Output
   const input = [1, 2, 3, 4, 5, 6, 3, 2, 7, 8, 6];
   const output = findDuplicates(input);

   console.log('Input:', input); // Input: [1, 2, 3, 4, 5, 6, 3, 2, 7, 8, 6]
   console.log('Output:', output); // Output: [3, 2, 6]
   ```

4. **Find Missing Number in Array**

   ```js
   function findMissingNumber(arr) {
     const n = arr.length + 1; // because one number is missing
     const expectedSum = (n * (n + 1)) / 2; // sum of first n natural numbers
     let actualSum = 0;

     // Manually sum the array elements
     for (let i = 0; i < arr.length; i++) {
       actualSum += arr[i];
     }

     return expectedSum - actualSum; // the missing number
   }

   // Example Input and Output

   const input = [1, 2, 4, 5, 6]; // Expected output: 3
   const output = findMissingNumber(input);
   console.log(output); // Output: 3
   ```

5. **Check Anagram**

   ```js
   function areAnagrams(str1, str2) {
     // Step 1: Check if the lengths are the same
     if (str1.length !== str2.length) {
       return false;
     }

     // Step 2: Sort both strings and compare
     const sortedStr1 = str1.toLowerCase().split('').sort().join('');
     const sortedStr2 = str2.toLowerCase().split('').sort().join('');

     return sortedStr1 === sortedStr2;
   }

   // Example input and output
   const str1 = 'listen';
   const str2 = 'silent';

   console.log(areAnagrams(str1, str2)); // Output: true
   ```

6. **Maximum Subarray Sum (Kadane’s Algorithm)**

   ```js
   function maxSubArray(arr) {
     let maxSum = arr[0];
     let currentSum = arr[0];
     for (let i = 1; i < arr.length; i++) {
       currentSum = Math.max(arr[i], currentSum + arr[i]);
       maxSum = Math.max(maxSum, currentSum);
     }
     return maxSum;
   }

   // Example Input and Output
   const input = [-2, 1, -3, 4, -1, 2, 1, -5, 4];
   const output = maxSubArray(input);

   console.log('Input:', input);
   // Input: [-2, 1, -3, 4, -1, 2, 1, -5, 4]
   console.log('Output:', output);
   // Output: 6
   ```

7. **Find All Pairs with a Given Sum**

   ```js
   function findPairs(arr, target) {
     const pairs = [];
     const set = new Set();
     for (let num of arr) {
       const complement = target - num;
       if (set.has(complement)) pairs.push([num, complement]);
       set.add(num);
     }
     return pairs;
   }

   // Find all unique pairs in an array whose sum is equal to a given target number.
   // Input : findPairs([1, 2, 3, 4, 5], 5);
   // Output : [[3, 2], [4, 1]]
   ```

8. **Longest Common Prefix**

   ```js
   function longestCommonPrefix(strs) {
     if (!strs.length) return '';
     for (let i = 0; i < strs[0].length; i++) {
       for (let str of strs) {
         if (str[i] !== strs[0][i]) return str.slice(0, i);
       }
     }
     return strs[0];
   }

   // Find the longest common starting substring among an array of strings.
   // Input : longestCommonPrefix(["flower", "flow", "flight"]);
   // Output : "fl"
   ```

9. **Move Zeros to End**

   ```js
   function moveZeroes(nums) {
     let i = 0;
     for (let j = 0; j < nums.length; j++) {
       if (nums[j] !== 0) [nums[i++], nums[j]] = [nums[j], nums[i]];
     }
     return nums;
   }

   // Move all zeros in an array to the end while maintaining the order of non-zero elements.
   // Input : moveZeroes([0, 1, 0, 3, 12]);
   // Output : [1, 3, 12, 0, 0]
   ```

10. **Remove Duplicates from Sorted Array**

```js
function removeDuplicates(nums) {
  let i = 0;
  for (let j = 1; j < nums.length; j++) {
    if (nums[i] !== nums[j]) nums[++i] = nums[j];
  }
  return nums.slice(0, i + 1);
}

// Remove duplicate elements from a sorted array and return the new array with unique elements.
// Input : removeDuplicates([0, 0, 1, 1, 1, 2, 2, 3, 3, 4]);
// Output : [0, 1, 2, 3, 4]
```

## Linked List

11. **Reverse a Linked List**

```js
function reverseList(head) {
  let prev = null,
    current = head;
  while (current) {
    [current.next, prev, current] = [prev, current, current.next];
  }
  return prev;
}
```

12. **Detect Cycle in a Linked List**

```js
function hasCycle(head) {
  let slow = head,
    fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }
  return false;
}
```

13. **Merge Two Sorted Linked Lists**

```js
function mergeTwoLists(l1, l2) {
  if (!l1 || !l2) return l1 || l2;
  if (l1.val < l2.val) {
    l1.next = mergeTwoLists(l1.next, l2);
    return l1;
  }
  l2.next = mergeTwoLists(l1, l2.next);
  return l2;
}
```

14. **Find Middle of Linked List**

```js
function findMiddle(head) {
  let slow = head,
    fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  return slow;
}
```

15. **Remove Nth Node from End**

```js
function removeNthFromEnd(head, n) {
  let dummy = { next: head };
  let fast = dummy,
    slow = dummy;
  for (let i = 0; i <= n; i++) fast = fast.next;
  while (fast) {
    fast = fast.next;
    slow = slow.next;
  }
  slow.next = slow.next.next;
  return dummy.next;
}
```

## Stack and Queue

16. **Implement Stack Using Array**

```js
class Stack {
  constructor() {
    this.stack = [];
  }
  push(value) {
    this.stack.push(value);
  }
  pop() {
    return this.stack.pop();
  }
  peek() {
    return this.stack[this.stack.length - 1];
  }
  isEmpty() {
    return this.stack.length === 0;
  }
}
```

17. **Implement Queue Using Array**

```js
class Queue {
  constructor() {
    this.queue = [];
  }
  enqueue(value) {
    this.queue.push(value);
  }
  dequeue() {
    return this.queue.shift();
  }
  front() {
    return this.queue[0];
  }
  isEmpty() {
    return this.queue.length === 0;
  }
}
```

18. **Valid Parentheses**

```js
function isValid(s) {
  const stack = [];
  const map = { '(': ')', '{': '}', '[': ']' };
  for (let char of s) {
    if (map[char]) stack.push(char);
    else if (stack.length === 0 || map[stack.pop()] !== char) return false;
  }
  return stack.length === 0;
}
```

19. **Min Stack**

```js
class MinStack {
  constructor() {
    this.stack = [];
  }
  push(val) {
    const min = this.stack.length === 0 ? val : Math.min(val, this.getMin());
    this.stack.push({ val, min });
  }
  pop() {
    this.stack.pop();
  }
  top() {
    return this.stack[this.stack.length - 1].val;
  }
  getMin() {
    return this.stack[this.stack.length - 1].min;
  }
}
```

20. **Implement Queue Using Stacks**

```js
class MyQueue {
  constructor() {
    this.inStack = [];
    this.outStack = [];
  }
  push(x) {
    this.inStack.push(x);
  }
  pop() {
    if (!this.outStack.length)
      while (this.inStack.length) this.outStack.push(this.inStack.pop());
    return this.outStack.pop();
  }
  peek() {
    return this.outStack[this.outStack.length - 1] || this.inStack[0];
  }
  empty() {
    return !this.inStack.length && !this.outStack.length;
  }
}
```

## Sorting and Searching

21. **Binary Search**

```js
function binarySearch(arr, target) {
  let left = 0,
    right = arr.length - 1;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) return mid;
    else if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  return -1;
}
```

22. **Linear Searching**

```js
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) return i;
  }
  return -1;
}

// Find the index of a target element in an array.
// Input: linearSearch([1, 2, 3, 4, 5], 4);
// Output: 3
```

23. **Bubble Sort**

```js
function bubbleSort(arr) {
  let n = arr.length;
  for (let i = 0; i < n - 1; i++) {
    for (let j = 0; j < n - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
      }
    }
  }
  return arr;
}

// Sort an array using bubble sort algorithm.
// Input: bubbleSort([5, 1, 4, 2, 8]);
// Output: [1, 2, 4, 5, 8]
```

24. **Selection Sort**

```js
function selectionSort(arr) {
  let n = arr.length;
  for (let i = 0; i < n - 1; i++) {
    let minIndex = i;
    for (let j = i + 1; j < n; j++) {
      if (arr[j] < arr[minIndex]) minIndex = j;
    }
    [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
  }
  return arr;
}

// Sort an array by repeatedly finding the minimum element and moving it to the beginning.
// Input: selectionSort([64, 25, 12, 22, 11]);
// Output: [11, 12, 22, 25, 64]
```

25. **Insertion Sort**

```js
function insertionSort(arr) {
  let n = arr.length;
  for (let i = 1; i < n; i++) {
    let key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
  }
  return arr;
}

// Build the sorted array one item at a time by inserting elements into their correct position.
// Input: insertionSort([12, 11, 13, 5, 6]);
// Output: [5, 6, 11, 12, 13]
```

25. **Insertion Sort**

```js
function insertionSort(arr) {
  let n = arr.length;
  for (let i = 1; i < n; i++) {
    let key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
  }
  return arr;
}

// Build the sorted array one item at a time by inserting elements into their correct position.
// Input: insertionSort([12, 11, 13, 5, 6]);
// Output: [5, 6, 11, 12, 13]
```

🧵 Top 25 String Coding Questions

- Reverse a string
- Check if a string is a palindrome
- Find the first non-repeating character
- Check if two strings are anagrams
- Count and say problem
- Longest common prefix among an array of strings
- String compression (e.g., aabcccccaaa → a2b1c5a3)
- Implement strstr() / indexOf()
- Longest substring without repeating characters
- Longest palindromic substring
- Group anagrams together
- Check if a string is a valid palindrome (ignore spaces and symbols)
- Find all permutations of a string
- Minimum number of insertions to make a string palindrome
- Check if one string is a rotation of another
- Roman to Integer and Integer to Roman
- ZigZag string conversion
- Decode a string (e.g., '3[a2[c]]' → 'accaccacc')
- Word Break Problem
- Compare version numbers
- Multiply two large numbers represented as strings
- Remove duplicate letters to make smallest lexicographical string
- Minimum window substring (substring containing all characters of another string)
- Check if two strings are isomorphic
- Wildcard matching with ‘\*’ and ‘?’

📦 Top 25 Array Coding Questions

- Find the maximum and minimum element in an array
- Reverse an array
- Find the "Kth" largest/smallest element
- Sort an array of 0s, 1s, and 2s (Dutch National Flag problem)
- Move all zeros to the end
- Left rotate an array by one place / by 'd' places
- Find the union and intersection of two arrays
- Cyclically rotate an array by one
- Kadane’s Algorithm (maximum subarray sum)
- Find the missing number in an array (1 to n)
- Find the duplicate element
- Merge two sorted arrays
- Merge intervals
- Stock buy and sell to maximize profit
- Find the majority element (more than n/2 times)
- Best time to buy and sell stock (Single Transaction)
- Trapping Rain Water
- Subarray with given sum (positive numbers)
- Longest Consecutive Subsequence
- Find all pairs with a given sum
- Find common elements in three sorted arrays
- Count inversions in an array
- Product of array elements except itself (without division)
- Find the repeating and the missing number
- Two sum problem

🔢 Top 25 Number-Based Coding Questions

- Check if a number is prime
- Find the Greatest Common Divisor (GCD) and Least Common Multiple (LCM)
- Reverse digits of an integer
- Check if a number is a palindrome
- Count the number of digits in an integer
- Find the factorial of a number
- Check if a number is an Armstrong number
- Check if a number is a perfect number
- Sum of digits of a number
- Power of a number (x^n) using fast exponentiation
- Check if a number is a power of 2
- Find the nth Fibonacci number (both recursion and DP)
- Sieve of Eratosthenes (Find all primes up to N)
- Find the missing number in an array of size n-1 containing numbers from 1 to n
- Add two numbers without using '+' operator
- Count number of set bits (1s) in binary representation
- Find square root of a number (without using sqrt())
- Check if a number can be expressed as a sum of two squares
- Generate all prime numbers in a given range
- Find the nth Catalan number
- Check if a number is a happy number
- Convert a decimal number to binary, octal, hexadecimal
- Find the number of trailing zeros in factorial of a number
- Find GCD of array elements
- Find if the given number is a strong number (sum of factorial of digits = number)
