### Promise.all in js

- Promise.all is a method that takes an array of promises and returns a single promise that resolves when all the promises in the array resolve. If any promise in the array is rejected, the returned promise is rejected with that reason. It's useful when you want to execute multiple asynchronous tasks in parallel and wait for all of them to complete.

```javascript
const fetchData1 = () =>
  new Promise((resolve) => setTimeout(() => resolve('Data 1'), 1000));
const fetchData2 = () =>
  new Promise((resolve) => setTimeout(() => resolve('Data 2'), 2000));
const fetchData3 = () =>
  new Promise((resolve) => setTimeout(() => resolve('Data 3'), 1500));

async function fetchAllData() {
  try {
    const results = await Promise.all([
      fetchData1(),
      fetchData2(),
      fetchData3(),
    ]);
    console.log('All data fetched:', results); // ["Data 1", "Data 2", "Data 3"]
  } catch (error) {
    console.error('Error in fetching data:', error);
  }
}

fetchAllData();
```

### Difference between import and require in js

- syntax
- - import -> import moduleName from 'module';
- - require -> const moduleName = require('module');

- type
- - ES6 module syntax & commonjs syntax

- execution
- - Static (at compile time) Dynamic(at runtime)

- default exports
- - supports both default & named exports
- - supports only module.exports

- Uses
- - Modern es6 environments
- - Older versions (before es6 modules)

### difference between map & forEach loops in js

- it returns a new array with transformed elements
- it does not returns any anything and transforms the actual array

- it can be chained with methods like .filter() or .reduce()
- it cannot be chained

### how to optimize API

1. Use caching - implement caching mechanism like redis.
2. User pagination - use pagination for large datasets.
3. Asynchronous processing - use asynchronous approach for api & I/O operations.
4. Load balancing - distribute requests across servers using load balancers.
5. Compression: Enable compression to reduce response size, using some libraries.
6. Minimize Latency: Optimize database queries and reduce external API calls.

### how to optimize db performances

1. Indexing: Create indexes for frequently queried columns.
2. Query Optimization: Avoid SELECT \*, write specific queries.
3. Normalization/De-normalization: Use appropriate schemas based on use case.
4. Connection Pooling: Reduce the overhead of creating connections.
5. Caching: Use tools like Redis or Memcached for frequently accessed data.
6. Monitoring: Use tools like Logging or database-specific monitoring for identifying bottlenecks.
