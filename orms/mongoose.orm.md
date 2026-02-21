# Mongoose ORM Interview Questions & Answers

## 1. What is Mongoose?

**Answer:** Mongoose is an Object Data Modeling (ODM) library for MongoDB and Node.js. It provides a schema-based solution to model application data, built-in type casting, validation, query building, and business logic hooks.

**Key Features:**
- Schema definition
- Built-in validation
- Middleware (hooks)
- Query building
- Population (joins)
- Plugins
- Discriminators

```javascript
const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost:27017/myapp');
```

---

## 2. What is a Schema in Mongoose?

**Answer:** A Schema defines the structure of documents within a collection, including field types, validation rules, default values, and indexes.

```javascript
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  age: {
    type: Number,
    min: 0,
    max: 120
  },
  isActive: {
    type: Boolean,
    default: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

---

## 3. What is the difference between Schema and Model?

**Answer:**

| Schema | Model |
|--------|-------|
| Defines structure | Wrapper for Schema |
| Blueprint for documents | Constructor for documents |
| No database interaction | Interacts with database |
| Created with new Schema() | Created with mongoose.model() |

```javascript
// Schema - defines structure
const userSchema = new mongoose.Schema({
  name: String,
  email: String
});

// Model - interacts with database
const User = mongoose.model('User', userSchema);

// Using model
const user = new User({ name: 'John', email: 'john@example.com' });
await user.save();
```

---

## 4. What are Schema Types in Mongoose?

**Answer:** Mongoose supports various schema types:

```javascript
const schema = new mongoose.Schema({
  // Basic types
  string: String,
  number: Number,
  boolean: Boolean,
  date: Date,

  // Complex types
  buffer: Buffer,
  objectId: mongoose.Schema.Types.ObjectId,
  mixed: mongoose.Schema.Types.Mixed,
  array: [],
  arrayOfStrings: [String],

  // Nested objects
  nested: {
    field1: String,
    field2: Number
  },

  // Map
  map: Map,
  mapOfStrings: {
    type: Map,
    of: String
  },

  // Decimal128 (for precise decimals)
  price: mongoose.Schema.Types.Decimal128,

  // UUID
  uuid: mongoose.Schema.Types.UUID
});
```

---

## 5. How do you create and save a document?

**Answer:**

```javascript
// Method 1: Create instance then save
const user = new User({
  name: 'John Doe',
  email: 'john@example.com'
});
await user.save();

// Method 2: Create directly
const user = await User.create({
  name: 'John Doe',
  email: 'john@example.com'
});

// Method 3: Insert many
const users = await User.insertMany([
  { name: 'John', email: 'john@example.com' },
  { name: 'Jane', email: 'jane@example.com' }
]);

// Method 4: findOneAndUpdate with upsert
const user = await User.findOneAndUpdate(
  { email: 'john@example.com' },
  { name: 'John Doe' },
  { upsert: true, new: true }
);
```

---

## 6. What are the different query methods in Mongoose?

**Answer:**

```javascript
// Find all
const users = await User.find();

// Find with conditions
const activeUsers = await User.find({ isActive: true });

// Find one
const user = await User.findOne({ email: 'john@example.com' });

// Find by ID
const user = await User.findById('507f1f77bcf86cd799439011');

// Find with select (projection)
const users = await User.find().select('name email');
const users = await User.find().select('-password'); // exclude

// Find with limit and skip
const users = await User.find().limit(10).skip(20);

// Find with sort
const users = await User.find().sort({ createdAt: -1 }); // descending
const users = await User.find().sort('name'); // ascending

// Count documents
const count = await User.countDocuments({ isActive: true });

// Exists
const exists = await User.exists({ email: 'john@example.com' });

// Distinct
const emails = await User.distinct('email');
```

---

## 7. How do you update documents in Mongoose?

**Answer:**

```javascript
// Update one
const result = await User.updateOne(
  { email: 'john@example.com' },
  { $set: { name: 'John Smith' } }
);

// Update many
const result = await User.updateMany(
  { isActive: false },
  { $set: { status: 'inactive' } }
);

// Find and update
const user = await User.findByIdAndUpdate(
  userId,
  { name: 'John Smith' },
  { new: true } // return updated document
);

// Find one and update
const user = await User.findOneAndUpdate(
  { email: 'john@example.com' },
  { name: 'John Smith' },
  { new: true, runValidators: true }
);

// Update using document
const user = await User.findById(userId);
user.name = 'John Smith';
await user.save();

// Update with operators
await User.updateOne(
  { _id: userId },
  {
    $set: { name: 'John' },
    $inc: { age: 1 },
    $push: { hobbies: 'reading' },
    $pull: { tags: 'old' }
  }
);
```

---

## 8. How do you delete documents in Mongoose?

**Answer:**

```javascript
// Delete one
await User.deleteOne({ email: 'john@example.com' });

// Delete many
await User.deleteMany({ isActive: false });

// Find by ID and delete
const user = await User.findByIdAndDelete(userId);

// Find one and delete
const user = await User.findOneAndDelete({ email: 'john@example.com' });

// Remove (deprecated, use deleteOne/deleteMany)
await User.remove({ isActive: false }); // Don't use

// Delete all documents
await User.deleteMany({});
```

---

## 9. What is Validation in Mongoose?

**Answer:** Mongoose provides built-in and custom validation for schema fields.

```javascript
const userSchema = new mongoose.Schema({
  // Required
  name: {
    type: String,
    required: [true, 'Name is required']
  },

  // Min/Max for numbers
  age: {
    type: Number,
    min: [18, 'Must be at least 18'],
    max: [100, 'Must be less than 100']
  },

  // Enum
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },

  // Custom validator
  email: {
    type: String,
    validate: {
      validator: function(v) {
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v);
      },
      message: 'Invalid email format'
    }
  },

  // Async validator
  username: {
    type: String,
    validate: {
      validator: async function(v) {
        const count = await mongoose.models.User.countDocuments({ username: v });
        return count === 0;
      },
      message: 'Username already exists'
    }
  },

  // MinLength/MaxLength
  password: {
    type: String,
    minlength: [6, 'Password must be at least 6 characters'],
    maxlength: 50
  }
});
```

---

## 10. What are Mongoose Middleware (Hooks)?

**Answer:** Middleware are functions that run at specific stages of document lifecycle.

**Types:**
- Document middleware (save, validate, remove, updateOne, deleteOne)
- Query middleware (find, findOne, updateMany, etc.)
- Aggregate middleware
- Model middleware

```javascript
// Pre-save middleware
userSchema.pre('save', async function(next) {
  // Hash password before saving
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 10);
  }
  next();
});

// Post-save middleware
userSchema.post('save', function(doc, next) {
  console.log(`User ${doc.name} saved`);
  next();
});

// Pre-remove middleware
userSchema.pre('remove', async function(next) {
  // Delete related data
  await Post.deleteMany({ author: this._id });
  next();
});

// Query middleware
userSchema.pre('find', function() {
  this.where({ isDeleted: { $ne: true } });
});

userSchema.pre(/^find/, function() {
  this.populate('author');
});

// Post-find middleware
userSchema.post('find', function(docs) {
  console.log(`Found ${docs.length} documents`);
});

// Aggregate middleware
userSchema.pre('aggregate', function() {
  this.pipeline().unshift({ $match: { isActive: true } });
});
```

---

## 11. What is Population in Mongoose?

**Answer:** Population automatically replaces ObjectId references with actual documents from other collections.

```javascript
// Define schemas with references
const authorSchema = new mongoose.Schema({
  name: String,
  email: String
});

const postSchema = new mongoose.Schema({
  title: String,
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Author'
  },
  comments: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Comment'
  }]
});

const Author = mongoose.model('Author', authorSchema);
const Post = mongoose.model('Post', postSchema);

// Basic population
const post = await Post.findById(postId).populate('author');
console.log(post.author.name); // Populated author object

// Populate multiple fields
const post = await Post.findById(postId)
  .populate('author')
  .populate('comments');

// Populate with select
const post = await Post.findById(postId)
  .populate('author', 'name email'); // Only name and email

// Nested population
const post = await Post.findById(postId)
  .populate({
    path: 'comments',
    populate: {
      path: 'author',
      select: 'name'
    }
  });

// Populate with conditions
const post = await Post.findById(postId)
  .populate({
    path: 'comments',
    match: { approved: true },
    options: { limit: 5, sort: { createdAt: -1 } }
  });
```

---

## 12. What are Virtual Properties?

**Answer:** Virtual properties are document properties that are not stored in MongoDB but computed on the fly.

```javascript
const userSchema = new mongoose.Schema({
  firstName: String,
  lastName: String,
  birthDate: Date
});

// Virtual getter
userSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

// Virtual setter
userSchema.virtual('fullName').set(function(v) {
  const parts = v.split(' ');
  this.firstName = parts[0];
  this.lastName = parts[1];
});

// Virtual with computation
userSchema.virtual('age').get(function() {
  return Math.floor((Date.now() - this.birthDate) / (365.25 * 24 * 60 * 60 * 1000));
});

// Include virtuals in JSON
userSchema.set('toJSON', { virtuals: true });
userSchema.set('toObject', { virtuals: true });

// Virtual populate
userSchema.virtual('posts', {
  ref: 'Post',
  localField: '_id',
  foreignField: 'author'
});

const user = await User.findById(userId).populate('posts');
```

---

## 13. What are Schema Methods vs Static Methods vs Instance Methods?

**Answer:**

```javascript
// Instance methods (called on documents)
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// Usage
const user = await User.findOne({ email: 'john@example.com' });
const isMatch = await user.comparePassword('password123');

// Static methods (called on Model)
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email });
};

// Usage
const user = await User.findByEmail('john@example.com');

// Query methods (chainable)
userSchema.query.byName = function(name) {
  return this.where({ name: new RegExp(name, 'i') });
};

// Usage
const users = await User.find().byName('john');
```

---

## 14. How do you handle indexes in Mongoose?

**Answer:**

```javascript
const userSchema = new mongoose.Schema({
  email: {
    type: String,
    unique: true, // Creates unique index
    index: true   // Creates regular index
  },
  name: {
    type: String,
    index: true
  },
  location: {
    type: { type: String, default: 'Point' },
    coordinates: [Number]
  }
});

// Compound index
userSchema.index({ email: 1, name: 1 });

// Text index
userSchema.index({ name: 'text', bio: 'text' });

// Geospatial index
userSchema.index({ location: '2dsphere' });

// Sparse index
userSchema.index({ phoneNumber: 1 }, { sparse: true });

// TTL index (auto-delete after time)
userSchema.index({ createdAt: 1 }, { expireAfterSeconds: 3600 });

// Ensure indexes are created
await User.createIndexes();

// List indexes
const indexes = await User.collection.getIndexes();

// Drop index
await User.collection.dropIndex('email_1');

// Disable auto-index creation
userSchema.set('autoIndex', false);
```

---

## 15. What is the difference between save() and update()?

**Answer:**

| save() | update() |
|--------|----------|
| Runs validators | Doesn't run validators by default |
| Triggers middleware | Limited middleware support |
| Returns document | Returns write result |
| Slower | Faster |
| For single documents | For bulk operations |

```javascript
// save() - runs all middleware and validators
const user = await User.findById(userId);
user.name = 'John Smith';
await user.save(); // Triggers pre/post save hooks

// update() - bypasses some Mongoose features
await User.updateOne(
  { _id: userId },
  { name: 'John Smith' }
); // Faster but no save middleware

// To run validators with update
await User.updateOne(
  { _id: userId },
  { name: 'John Smith' },
  { runValidators: true }
);
```

---

## 16. How do you implement pagination in Mongoose?

**Answer:**

```javascript
// Basic pagination
const page = 2;
const limit = 10;
const skip = (page - 1) * limit;

const users = await User.find()
  .skip(skip)
  .limit(limit);

// Pagination with total count
async function paginate(page = 1, limit = 10) {
  const skip = (page - 1) * limit;

  const [data, total] = await Promise.all([
    User.find().skip(skip).limit(limit),
    User.countDocuments()
  ]);

  return {
    data,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit)
    }
  };
}

// Cursor-based pagination (better for large datasets)
async function cursorPaginate(cursor, limit = 10) {
  const query = cursor ? { _id: { $gt: cursor } } : {};

  const data = await User.find(query)
    .sort({ _id: 1 })
    .limit(limit + 1);

  const hasMore = data.length > limit;
  const results = hasMore ? data.slice(0, -1) : data;
  const nextCursor = hasMore ? results[results.length - 1]._id : null;

  return {
    data: results,
    nextCursor,
    hasMore
  };
}
```

---

## 17. What are Discriminators in Mongoose?

**Answer:** Discriminators allow you to have multiple models with overlapping schemas on the same collection.

```javascript
// Base schema
const eventSchema = new mongoose.Schema({
  title: String,
  date: Date
}, { discriminatorKey: 'kind' });

const Event = mongoose.model('Event', eventSchema);

// Discriminator schemas
const clickEventSchema = new mongoose.Schema({
  element: String,
  count: Number
});

const pageViewSchema = new mongoose.Schema({
  url: String,
  duration: Number
});

// Create discriminators
const ClickEvent = Event.discriminator('ClickEvent', clickEventSchema);
const PageView = Event.discriminator('PageView', pageViewSchema);

// Usage
await ClickEvent.create({
  title: 'Button Click',
  element: 'submit-btn',
  count: 1
});

await PageView.create({
  title: 'Home Page',
  url: '/home',
  duration: 5000
});

// Query all events
const events = await Event.find(); // Returns both types

// Query specific discriminator
const clicks = await ClickEvent.find();
```

---

## 18. How do you handle transactions in Mongoose?

**Answer:**

```javascript
// Start a session
const session = await mongoose.startSession();

try {
  // Start transaction
  session.startTransaction();

  // Operations
  const user = await User.create([{
    name: 'John',
    email: 'john@example.com'
  }], { session });

  await Account.updateOne(
    { userId: user[0]._id },
    { $inc: { balance: 100 } },
    { session }
  );

  // Commit transaction
  await session.commitTransaction();
  console.log('Transaction committed');
} catch (error) {
  // Rollback on error
  await session.abortTransaction();
  console.log('Transaction aborted:', error);
} finally {
  // End session
  session.endSession();
}

// Using withTransaction helper
await mongoose.connection.transaction(async (session) => {
  await User.create([{ name: 'John' }], { session });
  await Post.create([{ title: 'Post' }], { session });
});
```

---

## 19. What are Plugins in Mongoose?

**Answer:** Plugins are reusable schema enhancements that can be applied to multiple schemas.

```javascript
// Create plugin
function timestampPlugin(schema) {
  schema.add({
    createdAt: { type: Date, default: Date.now },
    updatedAt: { type: Date, default: Date.now }
  });

  schema.pre('save', function(next) {
    this.updatedAt = Date.now();
    next();
  });
}

// Apply to schema
userSchema.plugin(timestampPlugin);

// Plugin with options
function softDeletePlugin(schema, options) {
  schema.add({
    isDeleted: { type: Boolean, default: false },
    deletedAt: Date
  });

  schema.methods.softDelete = function() {
    this.isDeleted = true;
    this.deletedAt = new Date();
    return this.save();
  };

  if (options && options.hideDeleted) {
    schema.pre('find', function() {
      this.where({ isDeleted: false });
    });
  }
}

userSchema.plugin(softDeletePlugin, { hideDeleted: true });

// Apply globally
mongoose.plugin(timestampPlugin);

// Popular plugins
// - mongoose-paginate-v2
// - mongoose-unique-validator
// - mongoose-delete (soft delete)
// - mongoose-autopopulate
```

---

## 20. How do you handle schema versioning?

**Answer:**

```javascript
// Version key (default: __v)
const userSchema = new mongoose.Schema({
  name: String,
  email: String
}, {
  versionKey: '__v' // Default
});

// Disable version key
const schema = new mongoose.Schema({}, { versionKey: false });

// Custom version key
const schema = new mongoose.Schema({}, { versionKey: 'version' });

// Optimistic concurrency
const user = await User.findById(userId);
user.name = 'John';
await user.save(); // Increments __v

// Handle version conflicts
try {
  const user = await User.findById(userId);
  user.name = 'John';
  await user.save();
} catch (error) {
  if (error.name === 'VersionError') {
    // Handle version conflict
    console.log('Document was modified by another process');
  }
}
```

---

## 21. What is Lean Query?

**Answer:** Lean queries return plain JavaScript objects instead of Mongoose documents, improving performance.

```javascript
// Regular query (returns Mongoose document)
const user = await User.findById(userId);
console.log(user.fullName); // Can use virtuals and methods
await user.save(); // Can save

// Lean query (returns plain object)
const user = await User.findById(userId).lean();
console.log(user.fullName); // undefined - no virtuals
// user.save() // Error - not a Mongoose document

// When to use lean:
// - Read-only operations
// - Performance critical queries
// - Large datasets
// - API responses that just need data

// Lean with options
const users = await User.find().lean({
  virtuals: true, // Include virtuals
  defaults: true  // Include default values
});

// Performance comparison
// Regular: ~100ms for 1000 docs
// Lean: ~10ms for 1000 docs
```

---

## 22. How do you implement full-text search?

**Answer:**

```javascript
// Create text index
const articleSchema = new mongoose.Schema({
  title: String,
  content: String,
  tags: [String]
});

articleSchema.index({ title: 'text', content: 'text' });

// Text search
const articles = await Article.find({
  $text: { $search: 'mongodb nodejs' }
});

// With score
const articles = await Article.find(
  { $text: { $search: 'mongodb' } },
  { score: { $meta: 'textScore' } }
).sort({ score: { $meta: 'textScore' } });

// Search with exact phrase
const articles = await Article.find({
  $text: { $search: '"exact phrase"' }
});

// Exclude words
const articles = await Article.find({
  $text: { $search: 'mongodb -mongoose' }
});

// Language-specific search
articleSchema.index({ content: 'text' }, { default_language: 'english' });
```

---

## 23. What are Subdocuments in Mongoose?

**Answer:**

```javascript
// Embedded subdocuments
const addressSchema = new mongoose.Schema({
  street: String,
  city: String,
  zipCode: String
});

const userSchema = new mongoose.Schema({
  name: String,
  address: addressSchema, // Single subdocument
  addresses: [addressSchema] // Array of subdocuments
});

const User = mongoose.model('User', userSchema);

// Create with subdocuments
const user = await User.create({
  name: 'John',
  address: {
    street: '123 Main St',
    city: 'NYC',
    zipCode: '10001'
  },
  addresses: [
    { street: '123 Main St', city: 'NYC' },
    { street: '456 Park Ave', city: 'LA' }
  ]
});

// Access subdocuments
console.log(user.address.city); // 'NYC'

// Modify subdocuments
user.address.city = 'Boston';
await user.save();

// Array subdocument operations
user.addresses.push({ street: '789 Oak Rd', city: 'SF' });
user.addresses[0].city = 'Chicago';
user.addresses.pull({ _id: addressId });
await user.save();

// Subdocument middleware
addressSchema.pre('save', function(next) {
  console.log('Address being saved');
  next();
});
```

---

## 24. How do you implement soft delete?

**Answer:**

```javascript
// Soft delete schema
const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  isDeleted: {
    type: Boolean,
    default: false
  },
  deletedAt: Date
});

// Soft delete method
userSchema.methods.softDelete = function() {
  this.isDeleted = true;
  this.deletedAt = new Date();
  return this.save();
};

// Restore method
userSchema.methods.restore = function() {
  this.isDeleted = false;
  this.deletedAt = null;
  return this.save();
};

// Query helper to exclude deleted
userSchema.query.active = function() {
  return this.where({ isDeleted: false });
};

// Automatically exclude deleted in queries
userSchema.pre(/^find/, function() {
  if (!this._skipSoftDelete) {
    this.where({ isDeleted: false });
  }
});

// Usage
const user = await User.findById(userId);
await user.softDelete();

// Find active users
const users = await User.find().active();

// Include deleted (skip filter)
const allUsers = await User.find()._skipSoftDelete = true;

// Using plugin: mongoose-delete
const mongooseDelete = require('mongoose-delete');
userSchema.plugin(mongooseDelete, {
  deletedAt: true,
  overrideMethods: 'all'
});
```

---

## 25. How do you handle aggregation in Mongoose?

**Answer:**

```javascript
// Basic aggregation
const result = await User.aggregate([
  { $match: { isActive: true } },
  { $group: { _id: '$role', count: { $sum: 1 } } },
  { $sort: { count: -1 } }
]);

// Complex aggregation pipeline
const stats = await Order.aggregate([
  // Match stage
  { $match: { status: 'completed' } },

  // Lookup (join)
  {
    $lookup: {
      from: 'users',
      localField: 'userId',
      foreignField: '_id',
      as: 'user'
    }
  },

  // Unwind
  { $unwind: '$user' },

  // Group
  {
    $group: {
      _id: '$user.country',
      totalOrders: { $sum: 1 },
      totalRevenue: { $sum: '$amount' },
      avgOrderValue: { $avg: '$amount' }
    }
  },

  // Project
  {
    $project: {
      country: '$_id',
      totalOrders: 1,
      totalRevenue: { $round: ['$totalRevenue', 2] },
      avgOrderValue: { $round: ['$avgOrderValue', 2] }
    }
  },

  // Sort
  { $sort: { totalRevenue: -1 } },

  // Limit
  { $limit: 10 }
]);

// Aggregation with pre hook
userSchema.pre('aggregate', function() {
  this.pipeline().unshift({ $match: { isDeleted: false } });
});

// Faceted aggregation
const result = await Product.aggregate([
  {
    $facet: {
      byCategory: [
        { $group: { _id: '$category', count: { $sum: 1 } } }
      ],
      priceRanges: [
        {
          $bucket: {
            groupBy: '$price',
            boundaries: [0, 50, 100, 200, 500],
            default: 'Other'
          }
        }
      ]
    }
  }
]);
```

---

## 26. What is the difference between findOneAndUpdate and updateOne?

**Answer:**

| findOneAndUpdate | updateOne |
|-----------------|-----------|
| Returns document | Returns write result |
| Atomic operation | Atomic operation |
| Can return old or new doc | Returns modification info |
| Slower | Faster |
| Good for single updates | Good for bulk updates |

```javascript
// findOneAndUpdate - returns document
const user = await User.findOneAndUpdate(
  { email: 'john@example.com' },
  { name: 'John Smith' },
  { new: true, runValidators: true }
);
console.log(user); // Updated user document

// updateOne - returns result
const result = await User.updateOne(
  { email: 'john@example.com' },
  { name: 'John Smith' }
);
console.log(result);
// {
//   acknowledged: true,
//   matchedCount: 1,
//   modifiedCount: 1,
//   upsertedCount: 0
// }
```

---

## 27. How do you implement data encryption in Mongoose?

**Answer:**

```javascript
// Install: npm install mongoose-field-encryption

const fieldEncryption = require('mongoose-field-encryption').fieldEncryption;

const userSchema = new mongoose.Schema({
  email: String,
  ssn: String,
  creditCard: String
});

// Encrypt specific fields
userSchema.plugin(fieldEncryption, {
  fields: ['ssn', 'creditCard'],
  secret: process.env.ENCRYPTION_KEY,
  saltGenerator: () => process.env.SALT
});

// Manual encryption with crypto
const crypto = require('crypto');

userSchema.pre('save', function(next) {
  if (this.isModified('ssn')) {
    const cipher = crypto.createCipher('aes-256-cbc', process.env.SECRET);
    let encrypted = cipher.update(this.ssn, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    this.ssn = encrypted;
  }
  next();
});

userSchema.methods.decryptSSN = function() {
  const decipher = crypto.createDecipher('aes-256-cbc', process.env.SECRET);
  let decrypted = decipher.update(this.ssn, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
};
```

---

## 28. How do you handle schema migration?

**Answer:**

```javascript
// Install: npm install migrate-mongoose

// migrations/001-add-status-field.js
module.exports = {
  async up() {
    await this.db.collection('users').updateMany(
      {},
      { $set: { status: 'active' } }
    );
  },

  async down() {
    await this.db.collection('users').updateMany(
      {},
      { $unset: { status: '' } }
    );
  }
};

// Manual migration
async function migrateUsers() {
  const users = await User.find({ oldField: { $exists: true } });

  for (const user of users) {
    user.newField = transformData(user.oldField);
    delete user.oldField;
    await user.save();
  }
}

// Schema versioning
const userSchema = new mongoose.Schema({
  name: String,
  schemaVersion: { type: Number, default: 1 }
});

userSchema.pre('save', async function() {
  if (this.schemaVersion < 2) {
    // Migrate to version 2
    this.newField = transformOldData(this);
    this.schemaVersion = 2;
  }
});
```

---

## 29. How do you implement caching with Mongoose?

**Answer:**

```javascript
// Redis caching
const redis = require('redis');
const client = redis.createClient();

// Cache wrapper
async function cachedQuery(key, ttl, queryFn) {
  // Check cache
  const cached = await client.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  // Execute query
  const result = await queryFn();

  // Cache result
  await client.setex(key, ttl, JSON.stringify(result));

  return result;
}

// Usage
const users = await cachedQuery(
  'users:all',
  3600,
  () => User.find().lean()
);

// Mongoose cache plugin
function cachePlugin(schema) {
  schema.statics.cachedFind = async function(conditions, ttl = 3600) {
    const key = `${this.modelName}:${JSON.stringify(conditions)}`;

    return cachedQuery(
      key,
      ttl,
      () => this.find(conditions).lean()
    );
  };
}

userSchema.plugin(cachePlugin);

// Clear cache on update
userSchema.post('save', async function() {
  await client.del(`User:*`);
});

// Using mongoose-cache
const mongoose = require('mongoose');
require('mongoose-cache').install(mongoose, {
  max: 50,
  maxAge: 1000 * 60
});

const users = await User.find().cache(60);
```

---

## 30. What are the best practices for Mongoose?

**Answer:**

**1. Always use lean() for read-only queries**
```javascript
const users = await User.find().lean();
```

**2. Use select() to limit fields**
```javascript
const users = await User.find().select('name email');
```

**3. Create indexes for frequent queries**
```javascript
userSchema.index({ email: 1 });
```

**4. Use transactions for related operations**
```javascript
const session = await mongoose.startSession();
session.startTransaction();
```

**5. Validate data properly**
```javascript
const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    validate: {
      validator: function(v) {
        return /^\S+@\S+\.\S+$/.test(v);
      }
    }
  }
});
```

**6. Handle connection errors**
```javascript
mongoose.connection.on('error', err => {
  console.error('MongoDB error:', err);
});
```

**7. Use aggregation for complex queries**
**8. Implement proper error handling**
**9. Don't mix callbacks and promises**
**10. Use connection pooling**

---

## 31. How do you implement audit logging?

**Answer:**

```javascript
const auditSchema = new mongoose.Schema({
  modelName: String,
  documentId: mongoose.Schema.Types.ObjectId,
  action: String,
  changes: mongoose.Schema.Types.Mixed,
  userId: mongoose.Schema.Types.ObjectId,
  timestamp: { type: Date, default: Date.now }
});

const Audit = mongoose.model('Audit', auditSchema);

// Audit plugin
function auditPlugin(schema) {
  schema.post('save', async function() {
    await Audit.create({
      modelName: this.constructor.modelName,
      documentId: this._id,
      action: 'create',
      changes: this.toObject()
    });
  });

  schema.post('findOneAndUpdate', async function(doc) {
    await Audit.create({
      modelName: doc.constructor.modelName,
      documentId: doc._id,
      action: 'update',
      changes: this.getUpdate()
    });
  });
}

userSchema.plugin(auditPlugin);
```

---

## 32. How do you handle file uploads with Mongoose?

**Answer:**

```javascript
// Store file metadata
const fileSchema = new mongoose.Schema({
  filename: String,
  originalName: String,
  mimeType: String,
  size: Number,
  path: String,
  uploadDate: { type: Date, default: Date.now }
});

// Using GridFS for large files
const Grid = require('gridfs-stream');
const conn = mongoose.createConnection(mongoURI);

let gfs;
conn.once('open', () => {
  gfs = Grid(conn.db, mongoose.mongo);
  gfs.collection('uploads');
});

// Upload file
const writeStream = gfs.createWriteStream({
  filename: file.name,
  mode: 'w',
  content_type: file.mimetype
});

fs.createReadStream(file.path).pipe(writeStream);

// Download file
const readStream = gfs.createReadStream({
  filename: filename
});
readStream.pipe(res);

// Using mongoose-file-upload plugin
```

---

## 33. What is Change Streams in Mongoose?

**Answer:**

```javascript
// Watch for changes
const changeStream = User.watch();

changeStream.on('change', (change) => {
  console.log('Change detected:', change);

  switch (change.operationType) {
    case 'insert':
      console.log('New user:', change.fullDocument);
      break;
    case 'update':
      console.log('Updated fields:', change.updateDescription);
      break;
    case 'delete':
      console.log('Deleted user:', change.documentKey);
      break;
  }
});

// Watch specific operations
const pipeline = [
  { $match: { operationType: 'insert' } }
];

const changeStream = User.watch(pipeline);

// Watch with filters
const changeStream = User.watch([
  {
    $match: {
      'fullDocument.status': 'active'
    }
  }
]);

// Close change stream
changeStream.close();
```

---

## 34. How do you implement multi-tenancy?

**Answer:**

```javascript
// Approach 1: Database per tenant
function getTenantDB(tenantId) {
  const uri = `mongodb://localhost:27017/tenant_${tenantId}`;
  return mongoose.createConnection(uri);
}

// Approach 2: Collection per tenant
const UserModel = (tenantId) => {
  return mongoose.model(`User_${tenantId}`, userSchema);
};

// Approach 3: Shared collection with tenant field
const userSchema = new mongoose.Schema({
  tenantId: {
    type: mongoose.Schema.Types.ObjectId,
    required: true,
    index: true
  },
  name: String,
  email: String
});

// Middleware to filter by tenant
userSchema.pre(/^find/, function() {
  if (this.options.tenantId) {
    this.where({ tenantId: this.options.tenantId });
  }
});

// Usage
const users = await User.find({}, null, {
  tenantId: currentTenantId
});
```

---

## 35. How do you optimize Mongoose queries?

**Answer:**

```javascript
// 1. Use indexes
userSchema.index({ email: 1, status: 1 });

// 2. Use lean()
const users = await User.find().lean();

// 3. Select only needed fields
const users = await User.find().select('name email');

// 4. Use pagination
const users = await User.find().limit(20).skip(0);

// 5. Avoid N+1 queries with populate
const posts = await Post.find().populate('author');

// 6. Use aggregation for complex queries
const result = await User.aggregate([...]);

// 7. Batch operations
await User.insertMany(users);

// 8. Use connection pooling
mongoose.connect(uri, {
  poolSize: 10
});

// 9. Use explain() to analyze queries
const explain = await User.find().explain();

// 10. Cache frequently accessed data
```

---

## 36. How do you handle geospatial queries?

**Answer:**

```javascript
const placeSchema = new mongoose.Schema({
  name: String,
  location: {
    type: {
      type: String,
      enum: ['Point'],
      required: true
    },
    coordinates: {
      type: [Number],
      required: true
    }
  }
});

// Create 2dsphere index
placeSchema.index({ location: '2dsphere' });

const Place = mongoose.model('Place', placeSchema);

// Find places near a location
const places = await Place.find({
  location: {
    $near: {
      $geometry: {
        type: 'Point',
        coordinates: [longitude, latitude]
      },
      $maxDistance: 5000 // 5km
    }
  }
});

// Find within radius
const places = await Place.find({
  location: {
    $geoWithin: {
      $centerSphere: [[longitude, latitude], radius / 6378.1] // km to radians
    }
  }
});

// Find within polygon
const places = await Place.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: 'Polygon',
        coordinates: [[
          [lng1, lat1],
          [lng2, lat2],
          [lng3, lat3],
          [lng1, lat1]
        ]]
      }
    }
  }
});
```

---

## 37. What is Model.bulkWrite()?

**Answer:**

```javascript
// Perform multiple write operations
const result = await User.bulkWrite([
  // Insert one
  {
    insertOne: {
      document: {
        name: 'John',
        email: 'john@example.com'
      }
    }
  },

  // Update one
  {
    updateOne: {
      filter: { email: 'jane@example.com' },
      update: { $set: { name: 'Jane Doe' } }
    }
  },

  // Update many
  {
    updateMany: {
      filter: { status: 'inactive' },
      update: { $set: { isDeleted: true } }
    }
  },

  // Replace one
  {
    replaceOne: {
      filter: { email: 'bob@example.com' },
      replacement: { name: 'Bob', email: 'bob@example.com', age: 30 }
    }
  },

  // Delete one
  {
    deleteOne: {
      filter: { email: 'old@example.com' }
    }
  },

  // Delete many
  {
    deleteMany: {
      filter: { lastLogin: { $lt: new Date('2020-01-01') } }
    }
  }
], {
  ordered: false // Continue on error
});

console.log(result);
// {
//   insertedCount: 1,
//   matchedCount: 2,
//   modifiedCount: 2,
//   deletedCount: 3,
//   upsertedCount: 0
// }
```

---

## 38. How do you implement data validation at multiple levels?

**Answer:**

```javascript
// Schema-level validation
const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: [true, 'Email is required'],
    unique: true,
    lowercase: true,
    trim: true,
    validate: {
      validator: function(v) {
        return /^\S+@\S+\.\S+$/.test(v);
      },
      message: 'Invalid email format'
    }
  },

  age: {
    type: Number,
    min: [18, 'Must be 18 or older'],
    max: [100, 'Invalid age'],
    validate: {
      validator: Number.isInteger,
      message: 'Age must be an integer'
    }
  },

  password: {
    type: String,
    required: true,
    minlength: 8,
    validate: {
      validator: function(v) {
        return /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(v);
      },
      message: 'Password must contain uppercase, lowercase, and number'
    }
  }
});

// Document-level validation
userSchema.pre('validate', function(next) {
  if (this.email && this.email.endsWith('@blocked.com')) {
    this.invalidate('email', 'Domain not allowed');
  }
  next();
});

// Custom validation method
userSchema.methods.validateAge = function() {
  if (this.age < 18) {
    throw new Error('User must be 18 or older');
  }
};

// Application-level validation with Joi
const Joi = require('joi');

const schema = Joi.object({
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(18).max(100),
  password: Joi.string().min(8).required()
});

const { error, value } = schema.validate(userData);
```

---

## 39. How do you handle schema relationships?

**Answer:**

```javascript
// One-to-One (Embedded)
const userSchema = new mongoose.Schema({
  name: String,
  profile: {
    bio: String,
    avatar: String
  }
});

// One-to-One (Reference)
const profileSchema = new mongoose.Schema({
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    unique: true
  },
  bio: String
});

// One-to-Many (Embedded)
const blogSchema = new mongoose.Schema({
  title: String,
  comments: [{
    text: String,
    author: String,
    date: Date
  }]
});

// One-to-Many (Reference - Child Referencing)
const postSchema = new mongoose.Schema({
  title: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }
});

// One-to-Many (Parent Referencing)
const authorSchema = new mongoose.Schema({
  name: String,
  posts: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Post'
  }]
});

// Many-to-Many
const studentSchema = new mongoose.Schema({
  name: String,
  courses: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Course'
  }]
});

const courseSchema = new mongoose.Schema({
  name: String,
  students: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Student'
  }]
});
```

---

## 40. What are common Mongoose pitfalls and how to avoid them?

**Answer:**

**1. Not waiting for connection**
```javascript
// Wrong
mongoose.connect(uri);
User.find(); // May fail

// Right
await mongoose.connect(uri);
const users = await User.find();
```

**2. Mixing callbacks and promises**
```javascript
// Wrong
User.find((err, users) => {
  // callback
}).then(users => {
  // promise
});

// Right - Use one approach
const users = await User.find();
```

**3. Not handling errors**
```javascript
// Wrong
const user = await User.findById(id);
console.log(user.name); // May throw if user is null

// Right
const user = await User.findById(id);
if (!user) {
  throw new Error('User not found');
}
```

**4. Using save() in loops**
```javascript
// Wrong - slow
for (const data of users) {
  await User.create(data);
}

// Right - fast
await User.insertMany(users);
```

**5. Not using lean() for read-only**
```javascript
// Wrong - slower
const users = await User.find();

// Right - faster
const users = await User.find().lean();
```

**6. Creating indexes in production**
```javascript
// Wrong - set in development only
userSchema.set('autoIndex', true);

// Right
userSchema.set('autoIndex', process.env.NODE_ENV === 'development');
```

**7. Not closing connections**
```javascript
// Always close when done
await mongoose.connection.close();
```

**8. Modifying _id**
```javascript
// Wrong - don't modify _id
user._id = newId;

// Right - create new document
```

**9. Not understanding middleware execution**
**10. Forgetting to add indexes for frequent queries**

