1. What is JavaScript?

- JavaScript is a high-level, versatile programming language primarily used to create dynamic and interactive elements on websites. It allows developers to implement complex features, such as interactive forms, animations, and content updates without reloading the page.

2. What is the role of JavaScript engines?

- JavaScript engines are programs or interpreters that execute JavaScript code. They convert JavaScript code into machine code that the computer's processor can execute. Modern JavaScript engines, like Google's V8 (used in Chrome) or Mozilla's SpiderMonkey (used in Firefox), optimize the execution of JavaScript code for better performance, enabling fast and efficient web applications.

3. what are client side and server side ?

- Client-side and server-side refer to the locations where operations or processes take place in a web application:

  - Client-side:

    - Refers to operations that are performed on the user's device, typically within a web browser.
    - Involves technologies like HTML, CSS, and JavaScript.
    - Examples include rendering web pages, validating form inputs, and handling user interactions without reloading the page.
    - Enhances user experience by providing immediate feedback and interactivity.

  - Server-side
    - Refers to operations that are performed on a remote server.
    - Involves server-side languages and frameworks like Node.js, PHP, Python, Ruby, or Java.
    - Examples include database interactions, user authentication, and processing business logic.
    - Ensures data integrity, security, and centralized processing, allowing for complex operations and data management.

4. what are variables ? what is the difference between let, const and var ?

- variables are used to store data, variables can hold various types of data, such as numbers, strings, objects, array and more.

- var :

  - Function-scoped: The scope is limited to the function where it is declared.
  - Can be redeclared and updated within its scope.
  - Hoisted to the top of its scope, meaning it can be used before its declaration, although its value will be `undefined` until assigned.

- let :

  - Block-scoped: The scope is limited to the block (e.g., `{}`) where it is declared.
  - Cannot be redeclared within the same scope but can be updated.
  - Not hoisted in the same way as `var`; using it before declaration results in a ReferenceError.

- const :

  - Block-scoped: The scope is limited to the block (e.g., `{}`) where it is declared.
  - Cannot be redeclared or updated within the same scope; it must be initialized at the time of declaration.
  - Not hoisted in the same way as `var`; using it before declaration results in a ReferenceError.
  - The value it holds cannot be changed, but if it holds an object or array, the properties or elements of that object or array can be changed.

5. what us DOM ? what is the difference between `HTML` and `DOM` ?

- the DOM(document object model) represent the web page as a tree like structure, that allows javascript to dynamically access and manipulate the content structure of webpage.

6. what are selectors in javascript ?

-
