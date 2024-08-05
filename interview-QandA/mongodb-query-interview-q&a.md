# MongoDB Query Notes

## Basic Queries

### Create

- Create documents in a collection:

```javascript
db.collection.insertOne({ key: value, key2: value2 });

db.collection.insertMany([
  { key: value1, key2: value2 },
  { key: value3, key2: value4 },
]);
```

### Find

- Find all documents in a collection:

```javascript
db.collection.find();

db.collection.find({ key: value });

db.collection.find({ key1: value1, key2: value2 });

db.collection.findOne({ key: value });
```

- Comparison Operators

```javascript
db.collection.find({ key: { $eq: value } });

db.collection.find({ key: { $ne: value } });

db.collection.find({ key: { $gt: value } });

db.collection.find({ key: { $gte: value } });

db.collection.find({ key: { $lt: value } });

db.collection.find({ key: { $lte: value } });
```

- Logical Operators

```javascript
db.collection.find({ $and: [{ key1: value1 }, { key2: value2 }] });

db.collection.find({ $or: [{ key1: value1 }, { key2: value2 }] });

db.collection.find({ key: { $not: { $eq: value } } });

db.collection.find({ $nor: [{ key1: value1 }, { key2: value2 }] });
```

- Element Operators

```javascript
db.collection.find({ key: { $exists: true } });

db.collection.find({ key: { $type: 'type' } });
```

- Evaluation Operators

```javascript
db.collection.find({ key: { $regex: /pattern/, $options: 'i' } });

db.collection.find({ $text: { $search: 'text' } });

db.collection.find({ key: { $mod: [divisor, remainder] } });
```

- Projection

```javascript
db.collection.find({}, { field1: 1, field2: 1 });

db.collection.find({ key: value }, { field1: 1, field2: 1 });

db.collection.find({ key: value }, { field1: 0, field2: 0 });
```

- Sorting

```javascript
db.collection.find().sort({ key: 1 }); // Ascending

db.collection.find().sort({ key: -1 }); // Descending
```

- Limiting and Skipping

```javascript
db.collection.find().limit(number);

db.collection.find().skip(number);
```

- Basic aggregation:

```javascript
db.collection.aggregate([
  { $match: { key: value } },
  { $group: { _id: '$key', total: { $sum: '$field' } } },
]);
```

- Project specific fields:

```javascript
db.collection.aggregate([{ $project: { key1: 1, key2: 1 } }]);
```

- Sort in aggregation:

```javascript
db.collection.aggregate([{ $sort: { key: 1 } }]);
```

### Update

- Update one document:

```javascript
db.collection.updateOne({ key: value }, { $set: { keyToUpdate: newValue } });
```

- Update multiple documents:

```javascript
db.collection.updateMany({ key: value }, { $set: { keyToUpdate: newValue } });
```

## Delete

- Delete one document:

```javascript
db.collection.deleteOne({ key: value });
```

- Delete multiple documents:

```javascript
db.collection.deleteMany({ key: value });
```

### Indexing

- Create an index:

```javascript
db.collection.createIndex({ key: 1 });
db.collection.createIndex({ key: 1 }, { unique: true });
```

### Miscellaneous

- Count documents:

```javascript
db.collection.countDocuments({ key: value });
```

- Distinct values:

```javascript
db.collection.distinct('key');
```

- Bulk Write:

```javascript
db.collection.bulkWrite([
  { insertOne: { document: { key: value } } },
  {
    updateOne: {
      filter: { key: value },
      update: { $set: { keyToUpdate: newValue } },
    },
  },
  { deleteOne: { filter: { key: value } } },
]);
```
