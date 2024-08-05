1. what is nodejs ?

- nodejs is neither a language nor a framework , nodejs is a run time environment for executing javascript code on server side.

2. how nodejs is a runtime environment on server side ? what is v8 ?

- nodejs use v8 engine internally to compiler js into machine code.
- browser executes javascript on the client side and every browser has its own js engine such as chrome as v8 engine firefox has spider monkey microsoft edge has chakra to compiler js into machine language.

3. what is difference between runtime environment & framework ?

- runtime environment primarily focuses on providing necessary infrastructure for code execution. where as framework primarily focuses on simplifying the development process by offering structured set of tools, libraries, and best practices.

4. what is difference between node.js and express.js ?

- nodejs is a runtime environment that allows to execute the js code in server side, whereas express.js is a framework built on top of the nodejs, it is designed to simplify the process of web applications and apis.

5. what are 7 main features fo nodejs ?

- single threaded, asynchronous, event-driven, v8 js engine, cross-platform, npm, real time capabilities.

6. what is single threaded programming ?

- Single-threaded programming is a programming paradigm where tasks are executed sequentially in a single thread of execution.

7. what is synchronous programming ?

- each task should be performed each after the other and the programs waits for each task to complete before moving on the next one.

8. what is multi-threaded programming ?

- Multi-threaded programming is the concurrent execution of multiple threads within a single process to perform tasks simultaneously.

9. what is asynchronous programming ?

- Asynchronous programming is a programming paradigm that allows tasks to run concurrently without blocking the execution of other tasks.

10. what is the difference between synchronous and asynchronous programming ?

- in synchronous programming : task are executed one after other
- in asynchronous programming : task are executed parallel order

- in synchronous programming : each task must complete before moving to next task
- in asynchronous programming : tasks can be executed independently of each other

- in synchronous programming : execution of task is blocked until a task is finished
- in asynchronous programming :execution of task is non blocking

11. what are events, event emitters, event queue, event loop & event driven

- Events: Actions or occurrences that happen in the system and can be handled by event handlers.
- Event Emitters: Objects in Node.js that facilitate communication by emitting named events and listening for them.
- Event Queue: A data structure that stores events to be processed by the event loop.
- Event Loop: A mechanism that handles and processes events and callbacks in Node.js, ensuring non-blocking I/O operations.
- Event-Driven: A programming paradigm where the flow of the program is determined by events, such as user actions or messages from other programs.

12. what are the main features and advantage

- Asynchronous and Event-Driven: Efficient handling of concurrent operations.
- Fast Execution: Powered by Google’s V8 engine for rapid code execution.
- Single Programming Language: JavaScript for both client and server sides.
- Rich Ecosystem: NPM offers a vast library of packages.
- Scalability: Suitable for building scalable network applications.

13. what are the main features and disadvantage

- Callback Hell/Complexity
- Single-Threaded Limits CPU-bound Tasks
- Immaturity of Some Libraries
- Performance Bottlenecks with Heavy Computation
- Limited Support for Multithreading

14. what is npm and node_modules ?

- npm (node package manager) is used to handle all the dependencies of the node project .
- node_modules is the folder which contains all the dependencies of the node project.

15. what is the role of package.json in a file ?

- package.json contains the metadata/information of the node project.

16. what are modules in node.js ?

- in programming language modules are a piece of functionality that can be easily reused in node project.

17. how many ways are there to export a module ?

- module.export.functionName = functionName;
- module.export = { functionName }

18. what will happen if we not export the module ?

- if you not export the module the module functions will not be available outside the module.

19. how to import singe and multiple function from a module ?

- const functionName = require('functionFile.js);
- const { functionName1, functionName2 } = require('functionFile.js);

20. what are module wrapper function ?

- each module is wrapped in a function called "module wrapper function" before it executes.

21. what are the types of module in node.js ?

- built in module / core module : these are the built in modules comes with nodejs
- - const http = require('http');
- - const fs = require('fs');
- - const path = require('path');

- local modules : these are the custom modules created by the user.

- third party modules : these modules are not the part of core modules but can be installed using packages managers such as npm, yarn pnpm and such.

22. what are the top 5 built in modules commonly used in nodejs projects ?

- fs module
- path module
- os module
- events module
- http module

23. explain - fs module, path module, os module, event module, http module.

- fs module : most important module in nodejs used to managing the files and folders, it provides the set of functions for interacting with the file system, ex - fs.readFile , fs.writeFile , fs.mkdir , fs.rmdir .. etc

- path module : path module provides utilities for joining, resolving, parsing, formatting, normalizing, and manipulating paths ex: path.join('/docs','file.txt'), path.parse('/src/docs/file.txt')

- os module : os module in nodejs provides us with the set of methods to interacts with operating system. ex- os.type // windows/linux, os.totalmem() // gets the total memory present in the os , ...

- event modules : use to register, create , or emit events while doing event driven programming.

- http module : http module can create a http server that listens to server port and gives a response back to the client.

24. what is the difference between event and functions in nodejs ?

- events are the actions that can be observed and responded to , events will call functions internally
- functions are the reusable piece of code that performs a specific task when invoked or called.

25. what is the use of createServer() method of http module ?

- it is use to create http server.

26. what are the advantages of using express.js with node.js ?

- simplified web development , express.js has easy syntax and more control for backend development using nodejs.
- easy integration of middleware functions into application request and response cycle.
- defines routes for handling http methods (GET, POST, PUT, DELETE, OTHERS .. etc)
- express.js supports many templates engines (handlebars, ejs ...) for generating dynamic server side content on server side.

27. what is middleware in express.js and when to use it ?

- a middleware in express.js is a function that handles http requests, performs operations, and passes control to the next middleware. ex- middle1 ( request logging ), middleware2 ( authentication ), middleware3 ( data validation ), middleware4 ( CORS ) and etc , these middleware are implemented in request pipeline.

28. what is the purpose of app.use() function in express.js

- app.use() method is used to execute(mount) middleware functions globally.

29. what is the purpose of next parameter in express.js ?

- the next parameter is the callback function which is used to pass control to the next middleware function in the stack.

30. what are the third party middlewares ? give some examples .

- third party middlewares are installed using npm, like helmet, body-parser, compression etc...
- we use it like app.use(helmet()), app.use(bodyParser.json()), app.use(compression()), etc...

31. can you summarize all types of middlewares ?

- application-level middlewares : middleware applies to all routes for logging , authentication etc..
- router level middlewares :
- built in middlewares :
- error handling middlewares :
- third party middlewares :

32. what are the advantages of middlewares ?

-

33. what is routing in express.js ?

-

34. https://www.youtube.com/watch?v=Nz-nPR5YJbw 2:20:00
