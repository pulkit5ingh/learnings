# MongoDB Aggregation Notes

### Aggregation Interview Questions and Answers

1. What is the Aggregation Framework in MongoDB?

- The Aggregation Framework is a feature in MongoDB that allows for data processing and transformation. It works through a series of pipeline stages where documents enter and undergo transformations to produce aggregated results.

2. Explain the $lookup stage in the Aggregation Framework.

- The $lookup stage performs a left outer join to another collection in the same database to filter in documents from the "joined" collection for processing.

3. How does $unwind work in MongoDB aggregation?

- The $unwind stage deconstructs an array field from the input documents to output a document for each element of the array. Each output document is a copy of the input document with the value of the array field replaced by a single element.

4. How would you calculate the total sales per product category?

- ```javascript
  db.sales.aggregate([
    { $group: { _id: '$category', totalSales: { $sum: '$amount' } } },
  ]);
  ```

5. What is the difference between $project and $addFields?

- $project is used to include, exclude, or add new fields to the documents, essentially reshaping them. $addFields is used to add new fields to the existing documents without removing any existing fields.

## Aggregation Framework

MongoDB's Aggregation Framework provides a way to process data and return computed results. It is based on data processing pipelines, where documents enter a multi-stage pipeline that transforms the documents into an aggregated result.

### Basic Aggregation

- Match stage to filter documents:

```javascript
db.collection.aggregate([{ $match: { key: value } }]);
```

- Group stage to group documents by a specific field:

```javascript
db.collection.aggregate([
  { $group: { _id: '$key', total: { $sum: '$field' } } },
]);
```

- Project stage to reshape documents and include/exclude fields:

```javascript
db.collection.aggregate([{ $project: { key1: 1, key2: 1 } }]);
```

- Sorting documents in aggregation:

```javascript
db.collection.aggregate([
  { $sort: { key: 1 } }, // 1 for ascending, -1 for descending
]);
```

- Limiting the number of documents:

```javascript
db.collection.aggregate([{ $limit: 10 }]);
```

- Skipping a number of documents:

```javascript
db.collection.aggregate([{ $skip: 5 }]);
```

- Unwind an array field to output a document for each element:

```javascript
db.collection.aggregate([{ $unwind: '$arrayField' }]);
```

- Lookup stage to perform a left outer join with another collection:

```javascript
db.collection.aggregate([
  {
    $lookup: {
      from: 'otherCollection',
      localField: 'key',
      foreignField: 'foreignKey',
      as: 'newField',
    },
  },
]);
```

- Replace root to replace the root document with a specified embedded document:

```javascript
db.collection.aggregate([{ $replaceRoot: { newRoot: '$embeddedDocument' } }]);
```

- Calculate the average of a field:

```javascript
db.collection.aggregate([
  { $group: { _id: null, averageValue: { $avg: '$field' } } },
]);
```

- Count documents in each group:

```javascript
db.collection.aggregate([{ $group: { _id: '$key', count: { $sum: 1 } } }]);
```

- Find the minimum value of a field:

```javascript
db.collection.aggregate([
  { $group: { _id: null, minValue: { $min: '$field' } } },
]);
```

- Concatenate fields:

```javascript
db.collection.aggregate([
  { $project: { fullName: { $concat: ['$firstName', ' ', '$lastName'] } } },
]);
```

- Find the total sales per product category for the last year, sorted by total sales:

```javascript
db.sales.aggregate([
  {
    $match: {
      date: {
        $gte: new ISODate('2023-01-01'),
        $lte: new ISODate('2023-12-31'),
      },
    },
  },
  { $group: { _id: '$category', totalSales: { $sum: '$amount' } } },
  { $sort: { totalSales: -1 } },
]);
```

- Calculate total sales only for categories that have more than 100 sales:

```javascript
db.sales.aggregate([
  {
    $group: {
      _id: '$category',
      totalSales: { $sum: '$amount' },
      count: { $sum: 1 },
    },
  },
  { $match: { count: { $gt: 100 } } },
  { $project: { _id: 0, category: '$_id', totalSales: 1 } },
]);
```

- Calculate the average score for each student:

```javascript
db.students.aggregate([
  { $unwind: '$scores' },
  { $group: { _id: '$studentId', averageScore: { $avg: '$scores.score' } } },
]);
```

- Nested Arrays and Unwind:

```javascript
db.students.aggregate([
  { $unwind: '$scores' },
  { $group: { _id: '$studentId', averageScore: { $avg: '$scores.score' } } },
]);
```

- Join the orders collection with the customers collection to get customer details with each order:

```javascript
db.orders.aggregate([
  {
    $lookup: {
      from: 'customers',
      localField: 'customerId',
      foreignField: 'customerId',
      as: 'customerDetails',
    },
  },
  { $unwind: '$customerDetails' },
]);
```
