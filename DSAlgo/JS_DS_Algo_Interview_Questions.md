https://linktr.ee/codeyatra
https://qset.io/

# JavaScript Data Structure and Algorithm Interview Questions

Here’s a curated list of 50 essential JavaScript-based data structure and algorithm questions, ideal for candidates with 2-5 years of experience.

## Number

1. **Check if the number is odd or even**

   **Happy Flow:**

   - Take a number as input
   - Use modulo operator (%) to check remainder when divided by 2
   - If remainder is 0, number is even
   - If remainder is 1, number is odd
   - Return appropriate string result

   ```js
   Input: 7;
   Output: Odd;

   Input: 8;
   Output: Even;

   function checkOddOrEven(number) {
     // Use modulo operator to check remainder when divided by 2
     // If remainder is 0, number is even; if 1, number is odd
     if (number % 2 === 0) {
       return 'Even';
     } else {
       return 'Odd';
     }
   }

   // Example usage:
   console.log(checkOddOrEven(7)); // Output: Odd
   console.log(checkOddOrEven(8)); // Output: Even
   console.log(checkOddOrEven(0)); // Output: Even
   console.log(checkOddOrEven(-3)); // Output: Odd
   ```

2. **Check if the number is prime**

   **Happy Flow:**

   - Check if number is ≤ 1 (not prime)
   - Check if number is 2 (only even prime)
   - Check if number is even > 2 (not prime)
   - Check divisors from 3 to √number (odd numbers only)
   - Return true if no divisors found

   ```js
   Input: 7;
   Output: true;

   Input: 8;
   Output: false;

   function isPrime(number) {
     // Edge cases: numbers ≤ 1 are not prime
     if (number <= 1) return false;

     // 2 is the only even prime number
     if (number === 2) return true;

     // Even numbers greater than 2 are not prime
     if (number % 2 === 0) return false;

     // Check odd divisors from 3 up to the square root
     // We only need to check up to √number because if n = a*b,
     // then at least one of a or b must be ≤ √n
     for (let i = 3; i <= Math.sqrt(number); i += 2) {
       if (number % i === 0) {
         return false; // Found a divisor, not prime
       }
     }

     return true; // No divisors found, it's prime
   }

   // Example usage:
   console.log(isPrime(7)); // Output: true
   console.log(isPrime(8)); // Output: false
   console.log(isPrime(17)); // Output: true
   console.log(isPrime(25)); // Output: false
   ```

3. **Check if the number is palindrome**

   **Happy Flow:**

   - Take a number as input
   - Convert number to string for easy manipulation
   - Reverse the string using split, reverse, and join
   - Compare original string with reversed string
   - Return true if they match, false otherwise

   ```js
   Input: 198;
   Output: False;

   Input: 121;
   Output: True;

   function isPalindrome(number) {
     // Convert number to string for easy string manipulation
     const str = number.toString();

     // Reverse the string: split into array, reverse, then join back
     const reversedStr = str.split('').reverse().join('');

     // Compare original string with reversed string
     return str === reversedStr;
   }

   // Example usage:
   console.log(isPalindrome(121)); // Output: true
   console.log(isPalindrome(198)); // Output: false
   console.log(isPalindrome(1221)); // Output: true
   console.log(isPalindrome(123)); // Output: false
   ```

4. **Check if two numbers have same last digits**

   **Happy Flow:**

   - Take two numbers as input
   - Extract last digit of each number using modulo 10
   - Use Math.abs() to handle negative numbers correctly
   - Compare the last digits
   - Return true if same, false otherwise

   ```js
   function haveSameLastDigit(num1, num2) {
     // Extract last digit using modulo 10
     // Math.abs() ensures we get the actual last digit even for negative numbers
     const lastDigit1 = Math.abs(num1) % 10;
     const lastDigit2 = Math.abs(num2) % 10;

     // Compare the last digits
     return lastDigit1 === lastDigit2;
   }

   // Example inputs and outputs
   console.log(haveSameLastDigit(27, 57)); // Output: true (both end with 7)
   console.log(haveSameLastDigit(123, 89)); // Output: false (3 and 9)
   console.log(haveSameLastDigit(-45, 5)); // Output: true (both end with 5)
   console.log(haveSameLastDigit(0, 10)); // Output: true (both end with 0)
   ```

5. **Round off a number to the next multiple of 5**

   **Happy Flow:**

   - Take a number as input
   - Check if number is already a multiple of 5
   - If yes, return the number as is
   - If no, calculate how much to add to reach next multiple
   - Add the difference to make it the next multiple of 5

   ```js
   Input: 46;
   Output: 50;

   Input: 21;
   Output: 25;

   Input: 30;
   Output: 30;

   function roundToNextMultipleOfFive(num) {
     // If number is already a multiple of 5, return as is
     if (num % 5 === 0) {
       return num;
     }

     // Calculate how much to add to reach the next multiple of 5
     // Formula: num + (5 - (num % 5))
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

6. **Convert a negative number to positive**

   **Happy Flow:**

   - Check if the number is negative
   - If negative, multiply by -1 to make it positive
   - If already positive or zero, return as is
   - Handle edge cases (zero, very large numbers)

   ```js
   Input: -5;
   Output: 5;

   Input: -2452;
   Output: 2452;

   function convertToPositive(number) {
     // If number is negative, multiply by -1 to make it positive
     // If number is already positive or zero, return as is
     return number < 0 ? number * -1 : number;
   }

   // Alternative using Math.abs() - cleaner approach
   function convertToPositive(number) {
     return Math.abs(number);
   }

   // Example usage:
   console.log(convertToPositive(-5)); // Output: 5
   console.log(convertToPositive(-2452)); // Output: 2452
   console.log(convertToPositive(10)); // Output: 10
   console.log(convertToPositive(0)); // Output: 0
   ```

7. **Find quotient and remainder by dividing an integer**

   **Happy Flow:**

   - Take dividend and divisor as input
   - Calculate quotient using Math.floor(dividend / divisor)
   - Calculate remainder using modulo operator (dividend % divisor)
   - Return both quotient and remainder
   - Handle edge cases (division by zero, negative numbers)

   ```js
   Input: divide 39 with 5;
   Output: quotient = 7 & remainder = 4;

   function divideWithRemainder(dividend, divisor) {
     // Handle division by zero
     if (divisor === 0) {
       throw new Error("Division by zero is not allowed");
     }

     // Calculate quotient (integer division)
     const quotient = Math.floor(dividend / divisor);

     // Calculate remainder
     const remainder = dividend % divisor;

     return { quotient, remainder };
   }

   // Alternative approach using built-in operators
   function divideWithRemainder(dividend, divisor) {
     if (divisor === 0) {
       throw new Error("Division by zero is not allowed");
     }

     return {
       quotient: Math.floor(dividend / divisor),
       remainder: dividend % divisor
     };
   }

   // Example usage:
   console.log(divideWithRemainder(39, 5));  // Output: { quotient: 7, remainder: 4 }
   console.log(divideWithRemainder(20, 3));  // Output: { quotient: 6, remainder: 2 }
   console.log(divideWithRemainder(15, 4));  // Output: { quotient: 3, remainder: 3 }
   ```

8. **Convert a Float Number to the Whole Number**

   **Happy Flow:**

   - Take a floating-point number as input
   - Use Math.floor() to truncate decimal part
   - Return the integer part only
   - Handle negative numbers correctly
   - Consider different rounding methods if needed

   ```js
   Input: 4.59;
   Output: 4;

   function convertFloatToWhole(floatNumber) {
     // Math.floor() truncates towards negative infinity
     // This removes the decimal part and returns the integer part
     return Math.floor(floatNumber);
   }

   // Alternative methods for different rounding behaviors:
   function convertFloatToWhole(floatNumber) {
     // Math.trunc() truncates towards zero (removes decimal part)
     return Math.trunc(floatNumber);
   }

   function convertFloatToWhole(floatNumber) {
     // Math.round() rounds to nearest integer
     return Math.round(floatNumber);
   }

   // Example usage:
   console.log(convertFloatToWhole(4.59)); // Output: 4
   console.log(convertFloatToWhole(4.99)); // Output: 4
   console.log(convertFloatToWhole(-2.7)); // Output: -3 (with Math.floor)
   console.log(convertFloatToWhole(7.0)); // Output: 7
   ```

9. **Check Whether a Number is Perfect Square**

   **Happy Flow:**

   - Take a number as input
   - Calculate square root using Math.sqrt()
   - Check if square root is an integer
   - Return true if perfect square, false otherwise
   - Handle edge cases (negative numbers, zero)

   ```js
   Input: 16;
   Output: 16 is perfect square: true;

   Input: 15;
   Output: 15 is perfect square: false;

   function isPerfectSquare(number) {
     // Negative numbers cannot be perfect squares
     if (number < 0) return false;

     // Calculate square root
     const sqrt = Math.sqrt(number);

     // Check if square root is an integer
     // Math.floor(sqrt) === sqrt ensures it's a whole number
     return Math.floor(sqrt) === sqrt;
   }

   // Alternative approach using modulo
   function isPerfectSquare(number) {
     if (number < 0) return false;

     const sqrt = Math.sqrt(number);
     // Check if sqrt is an integer by verifying no remainder
     return sqrt % 1 === 0;
   }

   // Example usage:
   console.log(isPerfectSquare(16));  // Output: true
   console.log(isPerfectSquare(15));  // Output: false
   console.log(isPerfectSquare(25));  // Output: true
   console.log(isPerfectSquare(0));   // Output: true
   console.log(isPerfectSquare(-4));  // Output: false
   ```

## Arrays and Strings

1. **Reverse a String**

   **Happy Flow:**

   - Take a string as input
   - Split the string into an array of characters
   - Reverse the array using reverse() method
   - Join the reversed array back into a string
   - Return the reversed string

   ```js
   // Function to reverse a string
   function reverseString(input) {
     // Split the string into an array of characters
     // Reverse the array order
     // Join the reversed array back into a string
     return input.split('').reverse().join('');
   }

   // Example usage:
   const input = 'hello';
   const output = reverseString(input);

   console.log('Input:', input); // Input: hello
   console.log('Output:', output); // Output: olleh

   // Additional examples:
   console.log(reverseString('world')); // Output: dlrow
   console.log(reverseString('12345')); // Output: 54321
   ```

2. **Check for Palindrome**

   **Happy Flow:**

   - Take a string as input
   - Clean the string (remove spaces, punctuation, convert to lowercase)
   - Create a reversed version of the cleaned string
   - Compare original cleaned string with reversed string
   - Return true if they match, false otherwise

   ```js
   // Function to check if a string is a palindrome
   function isPalindrome(str) {
     // Clean the string: convert to lowercase and remove non-alphanumeric characters
     const cleanStr = str.toLowerCase().replace(/[^a-z0-9]/g, '');

     // Compare the cleaned string with its reverse
     return cleanStr === cleanStr.split('').reverse().join('');
   }

   // Example Input and Output
   const input1 = 'A man, a plan, a canal: Panama';
   const input2 = 'hello';

   console.log(isPalindrome(input1)); // Output: true
   console.log(isPalindrome(input2)); // Output: false
   console.log(isPalindrome('racecar')); // Output: true
   console.log(isPalindrome('race a car')); // Output: false
   ```

3. **Find Duplicates in Array**

   **Happy Flow:**

   - Take an array as input
   - Initialize empty arrays for duplicates and visited elements
   - Iterate through each element
   - Skip if element already checked
   - For each element, check if it appears again later in array
   - Add to duplicates array if found and not already added
   - Mark element as visited
   - Return array of duplicate elements

   ```js
   function findDuplicates(arr) {
     const duplicates = []; // Store found duplicates
     const visited = []; // Track already checked elements

     for (let i = 0; i < arr.length; i++) {
       // Skip if the element is already checked to avoid redundant work
       if (visited.includes(arr[i])) continue;

       // Check for duplicates in the remaining part of array
       for (let j = i + 1; j < arr.length; j++) {
         if (arr[i] === arr[j] && !duplicates.includes(arr[i])) {
           duplicates.push(arr[i]);
           break; // Exit inner loop once duplicate is found
         }
       }

       visited.push(arr[i]); // Mark element as checked
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

   **Happy Flow:**

   - Take an array of consecutive numbers with one missing
   - Calculate expected sum using formula: n(n+1)/2
   - Calculate actual sum by iterating through array
   - Find missing number by subtracting actual sum from expected sum
   - Return the missing number

   ```js
   function findMissingNumber(arr) {
     // Array has n-1 elements, so total numbers should be n
     const n = arr.length + 1;

     // Sum of first n natural numbers: n(n+1)/2
     const expectedSum = (n * (n + 1)) / 2;

     // Calculate actual sum of array elements
     let actualSum = 0;
     for (let i = 0; i < arr.length; i++) {
       actualSum += arr[i];
     }

     // Missing number = expected sum - actual sum
     return expectedSum - actualSum;
   }

   // Example Input and Output
   const input = [1, 2, 4, 5, 6]; // Missing 3
   const output = findMissingNumber(input);

   console.log('Input:', input); // Input: [1, 2, 4, 5, 6]
   console.log('Missing number:', output); // Output: 3

   // Additional examples:
   console.log(findMissingNumber([1, 3, 4, 5])); // Output: 2
   console.log(findMissingNumber([2, 3, 4, 5])); // Output: 1
   ```

5. **Check Anagram**

   **Happy Flow:**

   - Take two strings as input
   - Check if lengths are equal (anagrams must have same length)
   - Convert both strings to lowercase for case-insensitive comparison
   - Sort characters in both strings
   - Compare sorted strings
   - Return true if identical, false otherwise

   ```js
   function areAnagrams(str1, str2) {
     // Step 1: Check if the lengths are the same
     // Anagrams must have the same number of characters
     if (str1.length !== str2.length) {
       return false;
     }

     // Step 2: Sort both strings and compare
     // Convert to lowercase for case-insensitive comparison
     // Split into array, sort, then join back to string
     const sortedStr1 = str1.toLowerCase().split('').sort().join('');
     const sortedStr2 = str2.toLowerCase().split('').sort().join('');

     // Compare sorted strings
     return sortedStr1 === sortedStr2;
   }

   // Example input and output
   const str1 = 'listen';
   const str2 = 'silent';

   console.log(areAnagrams(str1, str2)); // Output: true
   console.log(areAnagrams('hello', 'world')); // Output: false
   console.log(areAnagrams('evil', 'vile')); // Output: true
   ```

6. **Maximum Subarray Sum (Kadane's Algorithm)**

   **Happy Flow:**

   - Take an array as input
   - Initialize maxSum and currentSum with first element
   - Iterate through remaining elements
   - For each element, decide whether to start new subarray or extend current one
   - Update maxSum if current subarray sum is greater
   - Return the maximum subarray sum

   ```js
   function maxSubArray(arr) {
     // Initialize with first element
     let maxSum = arr[0];
     let currentSum = arr[0];

     // Iterate through remaining elements
     for (let i = 1; i < arr.length; i++) {
       // Either start new subarray or extend current one
       // Choose the maximum between current element alone or adding to existing sum
       currentSum = Math.max(arr[i], currentSum + arr[i]);

       // Update maximum sum found so far
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
   // Output: 6 (subarray [4, -1, 2, 1])
   ```

7. **Find All Pairs with a Given Sum**

   **Happy Flow:**

   - Take array and target sum as input
   - Initialize empty array for pairs and Set for tracking
   - Iterate through each number in array
   - Calculate complement (target - current number)
   - Check if complement exists in Set
   - If found, add pair to results and add current number to Set
   - Return all pairs found

   ```js
   function findPairs(arr, target) {
     const pairs = []; // Store all pairs that sum to target
     const set = new Set(); // Track numbers we've seen

     for (let num of arr) {
       // Calculate what number we need to pair with current number
       const complement = target - num;

       // If complement exists in our set, we found a pair
       if (set.has(complement)) {
         pairs.push([num, complement]);
       }

       // Add current number to set for future lookups
       set.add(num);
     }

     return pairs;
   }

   // Find all unique pairs in an array whose sum is equal to a given target number.
   // Input: findPairs([1, 2, 3, 4, 5], 5);
   // Output: [[3, 2], [4, 1]]

   console.log(findPairs([1, 2, 3, 4, 5], 5)); // Output: [[3, 2], [4, 1]]
   console.log(findPairs([2, 7, 11, 15], 9)); // Output: [[7, 2]]
   ```

8. **Longest Common Prefix**

   **Happy Flow:**

   - Take array of strings as input
   - Handle edge case: empty array returns empty string
   - Use first string as reference
   - Compare each character position across all strings
   - If any string differs at current position, return prefix up to that point
   - If all characters match, return the entire first string

   ```js
   function longestCommonPrefix(strs) {
     // Handle edge case: empty array
     if (!strs.length) return '';

     // Use first string as reference
     for (let i = 0; i < strs[0].length; i++) {
       // Check each string at position i
       for (let str of strs) {
         // If any string differs at position i, return prefix up to i
         if (str[i] !== strs[0][i]) {
           return strs[0].slice(0, i);
         }
       }
     }

     // If we reach here, first string is the common prefix
     return strs[0];
   }

   // Find the longest common starting substring among an array of strings.
   // Input: longestCommonPrefix(["flower", "flow", "flight"]);
   // Output: "fl"

   console.log(longestCommonPrefix(['flower', 'flow', 'flight'])); // Output: "fl"
   console.log(longestCommonPrefix(['dog', 'racecar', 'car'])); // Output: ""
   console.log(
     longestCommonPrefix(['interspecies', 'interstellar', 'interstate'])
   ); // Output: "inters"
   ```

9. **Move Zeros to End**

   **Happy Flow:**

   - Take array as input
   - Use two pointers: one for writing position, one for reading
   - Iterate through array with reading pointer
   - When non-zero element found, move it to writing position
   - Increment writing pointer after each move
   - Non-zero elements maintain their relative order
   - Zeros automatically end up at the end

   ```js
   function moveZeroes(nums) {
     let writeIndex = 0; // Position to write next non-zero element

     // First pass: move all non-zero elements to front
     for (let readIndex = 0; readIndex < nums.length; readIndex++) {
       if (nums[readIndex] !== 0) {
         // Swap current element with element at writeIndex
         [nums[writeIndex], nums[readIndex]] = [
           nums[readIndex],
           nums[writeIndex],
         ];
         writeIndex++; // Move write pointer forward
       }
     }

     return nums;
   }

   // Move all zeros in an array to the end while maintaining the order of non-zero elements.
   // Input: moveZeroes([0, 1, 0, 3, 12]);
   // Output: [1, 3, 12, 0, 0]

   console.log(moveZeroes([0, 1, 0, 3, 12])); // Output: [1, 3, 12, 0, 0]
   console.log(moveZeroes([0, 0, 1])); // Output: [1, 0, 0]
   ```

10. **Remove Duplicates from Sorted Array**

    **Happy Flow:**

- Take sorted array as input
- Use two pointers: one for unique elements, one for scanning
- Start with first element (always unique)
- Compare each element with previous unique element
- If different, move to next unique position
- Return array with only unique elements

```js
function removeDuplicates(nums) {
  if (nums.length === 0) return [];

  let uniqueIndex = 0; // Index for next unique element

  // Start from second element since first is always unique
  for (let scanIndex = 1; scanIndex < nums.length; scanIndex++) {
    // If current element is different from previous unique element
    if (nums[uniqueIndex] !== nums[scanIndex]) {
      uniqueIndex++; // Move to next unique position
      nums[uniqueIndex] = nums[scanIndex]; // Place unique element
    }
  }

  // Return array with only unique elements
  return nums.slice(0, uniqueIndex + 1);
}

// Remove duplicate elements from a sorted array and return the new array with unique elements.
// Input: removeDuplicates([0, 0, 1, 1, 1, 2, 2, 3, 3, 4]);
// Output: [0, 1, 2, 3, 4]

console.log(removeDuplicates([0, 0, 1, 1, 1, 2, 2, 3, 3, 4])); // Output: [0, 1, 2, 3, 4]
console.log(removeDuplicates([1, 1, 2])); // Output: [1, 2]
```

## Linked List

11. **Reverse a Linked List**

    **Happy Flow:**

- Take head of linked list as input
- Initialize prev as null, current as head
- Iterate through list while current exists
- Store next node before modifying current.next
- Reverse the link: current.next = prev
- Move prev to current, current to next
- Return prev as new head

```js
function reverseList(head) {
  let prev = null; // Previous node (initially null)
  let current = head; // Current node (start with head)

  // Iterate through the list
  while (current) {
    // Store next node before we lose the reference
    let next = current.next;

    // Reverse the link
    current.next = prev;

    // Move pointers forward
    prev = current;
    current = next;
  }

  // prev is now the new head
  return prev;
}

// Example usage:
// Input: 1 -> 2 -> 3 -> 4 -> null
// Output: 4 -> 3 -> 2 -> 1 -> null
```

12. **Detect Cycle in a Linked List**

    **Happy Flow:**

- Take head of linked list as input
- Use Floyd's cycle detection algorithm (tortoise and hare)
- Initialize slow and fast pointers at head
- Move slow pointer one step, fast pointer two steps
- If cycle exists, fast will eventually meet slow
- If fast reaches end, no cycle exists

```js
function hasCycle(head) {
  // Handle empty list or single node
  if (!head || !head.next) return false;

  let slow = head; // Tortoise - moves one step at a time
  let fast = head; // Hare - moves two steps at a time

  // Move pointers until fast reaches end or meets slow
  while (fast && fast.next) {
    slow = slow.next; // Move slow one step
    fast = fast.next.next; // Move fast two steps

    // If they meet, there's a cycle
    if (slow === fast) {
      return true;
    }
  }

  // Fast reached end, no cycle
  return false;
}

// Example usage:
// Input: 1 -> 2 -> 3 -> 2 (cycle back to 2)
// Output: true
```

13. **Merge Two Sorted Linked Lists**

    **Happy Flow:**

- Take two sorted linked lists as input
- Handle base cases: if one list is empty, return the other
- Compare values of current nodes from both lists
- Choose smaller value and recursively merge remaining lists
- Link the chosen node to result of recursive call
- Return the head of merged list

```js
function mergeTwoLists(l1, l2) {
  // Base cases: if one list is empty, return the other
  if (!l1) return l2;
  if (!l2) return l1;

  // Compare values and choose smaller one
  if (l1.val < l2.val) {
    // l1 is smaller, merge l1.next with l2
    l1.next = mergeTwoLists(l1.next, l2);
    return l1;
  } else {
    // l2 is smaller or equal, merge l1 with l2.next
    l2.next = mergeTwoLists(l1, l2.next);
    return l2;
  }
}

// Example usage:
// Input: [1,2,4] and [1,3,4]
// Output: [1,1,2,3,4,4]
```

14. **Find Middle of Linked List**

    **Happy Flow:**

- Take head of linked list as input
- Use Floyd's algorithm with slow and fast pointers
- Initialize both pointers at head
- Move slow one step, fast two steps
- When fast reaches end, slow is at middle
- Return slow pointer (middle node)

```js
function findMiddle(head) {
  if (!head) return null;

  let slow = head; // Moves one step at a time
  let fast = head; // Moves two steps at a time

  // Move fast pointer twice as fast as slow
  while (fast && fast.next) {
    slow = slow.next; // Move slow one step
    fast = fast.next.next; // Move fast two steps
  }

  // When fast reaches end, slow is at middle
  return slow;
}

// Example usage:
// Input: 1 -> 2 -> 3 -> 4 -> 5
// Output: Node with value 3

// Input: 1 -> 2 -> 3 -> 4
// Output: Node with value 3 (second middle for even length)
```

15. **Remove Nth Node from End**

    **Happy Flow:**

- Take head and n as input
- Create dummy node to handle edge cases
- Move fast pointer n+1 steps ahead
- Move both pointers until fast reaches end
- Slow pointer will be at node before target
- Remove target node by updating links
- Return head of modified list

```js
function removeNthFromEnd(head, n) {
  // Create dummy node to handle edge cases (like removing head)
  let dummy = { next: head };
  let fast = dummy;
  let slow = dummy;

  // Move fast pointer n+1 steps ahead
  for (let i = 0; i <= n; i++) {
    fast = fast.next;
  }

  // Move both pointers until fast reaches end
  while (fast) {
    fast = fast.next;
    slow = slow.next;
  }

  // Remove the nth node from end
  slow.next = slow.next.next;

  return dummy.next; // Return actual head
}

// Example usage:
// Input: [1,2,3,4,5], n = 2
// Output: [1,2,3,5] (removed 4th node from end)
```

## Stack and Queue

16. **Implement Stack Using Array**

    **Happy Flow:**

- Initialize empty array to store stack elements
- Push: Add element to end of array
- Pop: Remove and return last element
- Peek: Return last element without removing
- isEmpty: Check if array length is 0
- All operations are O(1) time complexity

```js
class Stack {
  constructor() {
    this.stack = []; // Array to store stack elements
  }

  // Add element to top of stack
  push(value) {
    this.stack.push(value);
  }

  // Remove and return top element
  pop() {
    if (this.isEmpty()) {
      throw new Error('Stack is empty');
    }
    return this.stack.pop();
  }

  // Return top element without removing
  peek() {
    if (this.isEmpty()) {
      throw new Error('Stack is empty');
    }
    return this.stack[this.stack.length - 1];
  }

  // Check if stack is empty
  isEmpty() {
    return this.stack.length === 0;
  }

  // Get stack size
  size() {
    return this.stack.length;
  }
}

// Example usage:
const stack = new Stack();
stack.push(1);
stack.push(2);
console.log(stack.peek()); // Output: 2
console.log(stack.pop()); // Output: 2
```

17. **Implement Queue Using Array**

    **Happy Flow:**

- Initialize empty array to store queue elements
- Enqueue: Add element to end of array
- Dequeue: Remove and return first element
- Front: Return first element without removing
- isEmpty: Check if array length is 0
- Note: shift() is O(n), but simple implementation

```js
class Queue {
  constructor() {
    this.queue = []; // Array to store queue elements
  }

  // Add element to rear of queue
  enqueue(value) {
    this.queue.push(value);
  }

  // Remove and return front element
  dequeue() {
    if (this.isEmpty()) {
      throw new Error('Queue is empty');
    }
    return this.queue.shift(); // O(n) operation
  }

  // Return front element without removing
  front() {
    if (this.isEmpty()) {
      throw new Error('Queue is empty');
    }
    return this.queue[0];
  }

  // Check if queue is empty
  isEmpty() {
    return this.queue.length === 0;
  }

  // Get queue size
  size() {
    return this.queue.length;
  }
}

// Example usage:
const queue = new Queue();
queue.enqueue(1);
queue.enqueue(2);
console.log(queue.front()); // Output: 1
console.log(queue.dequeue()); // Output: 1
```

18. **Valid Parentheses**

    **Happy Flow:**

- Take string of parentheses as input
- Initialize empty stack and mapping of opening to closing brackets
- Iterate through each character
- If opening bracket, push to stack
- If closing bracket, check if stack is empty or doesn't match
- If valid, pop from stack; if invalid, return false
- Return true if stack is empty at end

```js
function isValid(s) {
  const stack = [];
  // Mapping of opening brackets to closing brackets
  const map = { '(': ')', '{': '}', '[': ']' };

  for (let char of s) {
    // If character is an opening bracket
    if (map[char]) {
      stack.push(char);
    } else {
      // If character is a closing bracket
      // Check if stack is empty or brackets don't match
      if (stack.length === 0 || map[stack.pop()] !== char) {
        return false;
      }
    }
  }

  // Valid if all brackets are matched (stack is empty)
  return stack.length === 0;
}

// Example usage:
console.log(isValid('()')); // Output: true
console.log(isValid('()[]{}')); // Output: true
console.log(isValid('(]')); // Output: false
console.log(isValid('([)]')); // Output: false
```

19. **Min Stack**

    **Happy Flow:**

- Initialize stack to store objects with value and minimum
- Push: Store value and current minimum in each element
- Pop: Remove top element
- Top: Return value of top element
- GetMin: Return minimum value from top element
- All operations maintain minimum in O(1) time

```js
class MinStack {
  constructor() {
    this.stack = []; // Store objects with {val, min}
  }

  // Push element onto stack
  push(val) {
    // Calculate minimum: current val or previous minimum
    const min = this.stack.length === 0 ? val : Math.min(val, this.getMin());

    // Store both value and minimum
    this.stack.push({ val, min });
  }

  // Remove top element
  pop() {
    if (this.stack.length === 0) {
      throw new Error('Stack is empty');
    }
    this.stack.pop();
  }

  // Get top element value
  top() {
    if (this.stack.length === 0) {
      throw new Error('Stack is empty');
    }
    return this.stack[this.stack.length - 1].val;
  }

  // Get minimum element in O(1) time
  getMin() {
    if (this.stack.length === 0) {
      throw new Error('Stack is empty');
    }
    return this.stack[this.stack.length - 1].min;
  }
}

// Example usage:
const minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
console.log(minStack.getMin()); // Output: -3
minStack.pop();
console.log(minStack.top()); // Output: 0
console.log(minStack.getMin()); // Output: -2
```

20. **Implement Queue Using Stacks**

    **Happy Flow:**

- Use two stacks: inStack for enqueue, outStack for dequeue
- Enqueue: Push to inStack
- Dequeue: If outStack empty, transfer all from inStack to outStack, then pop
- Peek: Return top of outStack or first of inStack
- Empty: Check if both stacks are empty
- Amortized O(1) time complexity

```js
class MyQueue {
  constructor() {
    this.inStack = []; // Stack for enqueue operations
    this.outStack = []; // Stack for dequeue operations
  }

  // Add element to rear of queue
  push(x) {
    this.inStack.push(x);
  }

  // Remove and return front element
  pop() {
    // If outStack is empty, transfer all elements from inStack
    if (!this.outStack.length) {
      while (this.inStack.length) {
        this.outStack.push(this.inStack.pop());
      }
    }
    return this.outStack.pop();
  }

  // Return front element without removing
  peek() {
    // If outStack has elements, return top
    if (this.outStack.length) {
      return this.outStack[this.outStack.length - 1];
    }
    // Otherwise, return first element of inStack
    return this.inStack[0];
  }

  // Check if queue is empty
  empty() {
    return !this.inStack.length && !this.outStack.length;
  }
}

// Example usage:
const queue = new MyQueue();
queue.push(1);
queue.push(2);
console.log(queue.peek()); // Output: 1
console.log(queue.pop()); // Output: 1
console.log(queue.empty()); // Output: false
```

## Sorting and Searching

21. **Binary Search**

    **Happy Flow:**

- Take sorted array and target as input
- Initialize left and right pointers
- While left <= right, calculate middle index
- Compare middle element with target
- If equal, return middle index
- If target smaller, search left half
- If target larger, search right half
- Return -1 if not found

```js
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;

  // Continue searching while search space is valid
  while (left <= right) {
    // Calculate middle index (avoid overflow)
    const mid = Math.floor((left + right) / 2);

    // Check if target found
    if (arr[mid] === target) {
      return mid;
    } else if (arr[mid] < target) {
      // Target is in right half
      left = mid + 1;
    } else {
      // Target is in left half
      right = mid - 1;
    }
  }

  // Target not found
  return -1;
}

// Example usage:
const arr = [1, 3, 5, 7, 9, 11, 13];
console.log(binarySearch(arr, 7)); // Output: 3
console.log(binarySearch(arr, 4)); // Output: -1
```

22. **Linear Searching**

    **Happy Flow:**

- Take array and target as input
- Iterate through each element sequentially
- Compare current element with target
- If match found, return index
- If no match after checking all elements, return -1
- Time complexity: O(n)

```js
function linearSearch(arr, target) {
  // Iterate through each element
  for (let i = 0; i < arr.length; i++) {
    // If target found, return index
    if (arr[i] === target) {
      return i;
    }
  }

  // Target not found
  return -1;
}

// Find the index of a target element in an array.
// Input: linearSearch([1, 2, 3, 4, 5], 4);
// Output: 3

console.log(linearSearch([1, 2, 3, 4, 5], 4)); // Output: 3
console.log(linearSearch([1, 2, 3, 4, 5], 6)); // Output: -1
```

23. **Bubble Sort**

    **Happy Flow:**

- Take unsorted array as input
- Use nested loops: outer for passes, inner for comparisons
- Compare adjacent elements and swap if out of order
- After each pass, largest element "bubbles" to end
- Continue until array is sorted
- Time complexity: O(n²)

```js
function bubbleSort(arr) {
  const n = arr.length;

  // Outer loop: number of passes
  for (let i = 0; i < n - 1; i++) {
    // Inner loop: compare adjacent elements
    // After i passes, last i elements are already sorted
    for (let j = 0; j < n - i - 1; j++) {
      // Swap if current element is greater than next
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

console.log(bubbleSort([5, 1, 4, 2, 8])); // Output: [1, 2, 4, 5, 8]
console.log(bubbleSort([64, 34, 25, 12, 22, 11, 90])); // Output: [11, 12, 22, 25, 34, 64, 90]
```

24. **Selection Sort**

    **Happy Flow:**

- Take unsorted array as input
- Find minimum element in unsorted portion
- Swap minimum with first element of unsorted portion
- Move boundary of sorted portion one position right
- Repeat until entire array is sorted
- Time complexity: O(n²)

```js
function selectionSort(arr) {
  const n = arr.length;

  // Outer loop: move boundary of sorted array
  for (let i = 0; i < n - 1; i++) {
    // Find minimum element in unsorted portion
    let minIndex = i;
    for (let j = i + 1; j < n; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }

    // Swap minimum element with first element of unsorted portion
    [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
  }

  return arr;
}

// Sort an array by repeatedly finding the minimum element and moving it to the beginning.
// Input: selectionSort([64, 25, 12, 22, 11]);
// Output: [11, 12, 22, 25, 64]

console.log(selectionSort([64, 25, 12, 22, 11])); // Output: [11, 12, 22, 25, 64]
console.log(selectionSort([5, 2, 8, 1, 9])); // Output: [1, 2, 5, 8, 9]
```

25. **Insertion Sort**

    **Happy Flow:**

- Take unsorted array as input
- Start from second element (first is already "sorted")
- Pick current element as key
- Compare key with elements in sorted portion
- Shift larger elements one position right
- Insert key at correct position
- Repeat until entire array is sorted
- Time complexity: O(n²)

```js
function insertionSort(arr) {
  const n = arr.length;

  // Start from second element (first element is already sorted)
  for (let i = 1; i < n; i++) {
    let key = arr[i]; // Current element to be inserted
    let j = i - 1; // Index of last element in sorted portion

    // Move elements of sorted portion that are greater than key
    // one position to the right
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }

    // Insert key at correct position
    arr[j + 1] = key;
  }

  return arr;
}

// Build the sorted array one item at a time by inserting elements into their correct position.
// Input: insertionSort([12, 11, 13, 5, 6]);
// Output: [5, 6, 11, 12, 13]

console.log(insertionSort([12, 11, 13, 5, 6])); // Output: [5, 6, 11, 12, 13]
console.log(insertionSort([5, 2, 4, 6, 1, 3])); // Output: [1, 2, 3, 4, 5, 6]
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
