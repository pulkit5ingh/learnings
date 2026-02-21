# MongoDB Learning Roadmap

> Find the interactive version of this roadmap and more roadmaps at [roadmap.sh](https://roadmap.sh/mongodb)

## Related Roadmaps
- [Backend Roadmap](https://roadmap.sh/backend)
- [Node.js Roadmap](https://roadmap.sh/nodejs)
- [Frontend Roadmap](https://roadmap.sh/frontend)

---

## 1. MongoDB Basics

### SQL vs NoSQL
- Understanding relational vs non-relational databases
- Key differences and use cases
- Scalability and flexibility considerations

### What is MongoDB?
- Document-oriented NoSQL database
- Schema-less architecture
- Horizontal scalability
- High performance and availability

### When to use MongoDB?
- Flexible schema requirements
- Rapid development and prototyping
- Handling large volumes of unstructured data
- Real-time analytics and IoT applications
- Content management systems
- Scalable applications

### What is MongoDB Atlas?
- Fully managed cloud database service
- Automated backups and monitoring
- Multi-cloud support (AWS, Azure, GCP)
- Built-in security features

---

## 2. Data Model & Data Types

### MongoDB Terminology
- **Database**: Container for collections
- **Collection**: Group of MongoDB documents
- **Document**: Record in MongoDB (BSON format)
- **Field**: Key-value pair in a document

### BSON vs JSON
- **BSON** (Binary JSON): Binary representation of JSON
- More data types than JSON
- Faster to parse and traverse
- Includes type information

### Embedded Objects & Arrays
- Nested documents
- Array fields
- Denormalization strategies
- Document size limitations (16MB)

### Data Types

#### Primitive Types
- **String**: UTF-8 string data
- **Int32/Int**: 32-bit integer
- **Int64/Long**: 64-bit integer
- **Double**: 64-bit floating point
- **Decimal128**: High-precision decimal
- **Boolean**: true/false values

#### Complex Types
- **Object**: Embedded documents
- **Array**: List of values
- **Binary Data**: Binary data/blobs
- **Object ID**: Unique document identifier

#### Special Types
- **Date**: Date and time (milliseconds since Unix epoch)
- **Timestamp**: Internal MongoDB timestamp
- **Null**: Null value
- **Undefined**: Deprecated undefined value
- **Regular Expression**: Regex patterns
- **JavaScript**: JavaScript code
- **Symbol**: Deprecated symbol type
- **Min Key**: Lowest BSON value
- **Max Key**: Highest BSON value

---

## 3. Collections & Methods

### Document Operations

#### Insert Operations
- `insert()` - Insert single or multiple documents
- `insertOne()` - Insert a single document
- `insertMany()` - Insert multiple documents
- Batch inserts and performance considerations

#### Find Operations
- `find()` - Query documents
- `findOne()` - Find single document
- Query filters and projections
- Cursor methods: `limit()`, `skip()`, `sort()`

#### Update Operations
- `update()` - Update documents
- `updateOne()` - Update single document
- `updateMany()` - Update multiple documents
- `replaceOne()` - Replace entire document
- Update operators: `$set`, `$unset`, `$inc`, `$push`, `$pull`

#### Delete Operations
- `delete()` - Remove documents
- `deleteOne()` - Delete single document
- `deleteMany()` - Delete multiple documents

#### Other Methods
- `bulkWrite()` - Perform bulk write operations
- `validate()` - Validate collection structure
- `count()` / `countDocuments()` - Count documents
- `distinct()` - Get distinct values

---

## 4. Useful Concepts

### Cursors
- Iterator for query results
- Batch processing
- Cursor timeout and management
- `hasNext()`, `next()` methods

### Read / Write Concerns
- **Read Concern**: Consistency level for read operations
  - `local`, `available`, `majority`, `linearizable`, `snapshot`
- **Write Concern**: Acknowledgment level for write operations
  - `w`: Number of nodes to acknowledge
  - `j`: Journal acknowledgment
  - `wtimeout`: Timeout for write concern

### Retryable Reads / Writes
- Automatic retry on network errors
- Idempotent operations
- Fault tolerance

### Counting Documents
- `countDocuments()` - Accurate count
- `estimatedDocumentCount()` - Fast approximate count

---

## 5. Query Operators

### Projection Operators
- `$project` - Include/exclude fields
- `$include` - Include specific fields
- `$exclude` - Exclude specific fields
- `$slice` - Limit array elements

### Comparison Operators
- `$eq` - Equal to
- `$ne` - Not equal to
- `$gt` - Greater than
- `$gte` - Greater than or equal to
- `$lt` - Less than
- `$lte` - Less than or equal to

### Array Operators
- `$in` - Match any value in array
- `$nin` - Match none of the values
- `$all` - Match all values
- `$elemMatch` - Match array element conditions
- `$size` - Match array size

### Element Operators
- `$exists` - Field exists
- `$type` - Field type matching

### Logical Operators
- `$and` - Logical AND
- `$or` - Logical OR
- `$not` - Logical NOT
- `$nor` - Logical NOR

### Other Operators
- `$regex` - Regular expression matching
- `$text` - Text search
- `$where` - JavaScript expression

---

## 6. Performance Optimization

### Creating Indexes

#### Index Types
- **Single Field**: Index on single field
- **Compound**: Index on multiple fields
- **Text**: Full-text search index
- **Geospatial**: Location-based queries
- **Expiring (TTL)**: Auto-delete documents
- **Atlas Search Indexes**: Advanced search capabilities

#### Index Properties
- Unique indexes
- Partial indexes
- Sparse indexes
- Case-insensitive indexes

### Query Optimization
- Use `explain()` to analyze queries
- Index selection and strategy
- Query patterns and best practices
- Avoid full collection scans
- Proper use of projections
- Covered queries

---

## 7. Aggregation

### Pipelines, Stages and Operators
- Sequential data processing stages
- Pipeline optimization
- Early filtering with `$match`

### Common Operators

#### Pipeline Stages
- `$match` - Filter documents
- `$group` - Group by expression
- `$sort` - Sort documents
- `$project` - Reshape documents
- `$limit` - Limit results
- `$skip` - Skip documents
- `$unwind` - Deconstruct arrays
- `$lookup` - Join collections

#### Accumulator Operators
- `$sum` - Calculate sum
- `$avg` - Calculate average
- `$min` / `$max` - Min/max values
- `$first` / `$last` - First/last values
- `$push` - Create array of values
- `$addToSet` - Create array of unique values

---

## 8. Transactions

### ACID Properties
- Atomicity
- Consistency
- Isolation
- Durability

### Multi-Document Transactions
- Start and commit transactions
- Rollback on errors
- Transaction sessions
- Performance considerations

---

## 9. Developer Tools

### Language Drivers
- Official drivers for major languages
  - Node.js
  - Python (PyMongo)
  - Java
  - C#
  - Go
  - Ruby
  - PHP
- Driver features and best practices

### MongoDB Connectors
- **Kafka Connector**: Stream processing integration
- **Spark Connector**: Big data analytics
- **Elastic Search**: Full-text search integration
- BI Connectors
- Tableau, Power BI integration

---

## 10. Backup & Recovery

### Backup Methods
- `mongodump` - Binary export of data
- `mongorestore` - Restore from mongodump
- Cloud backups (Atlas)
- Filesystem snapshots
- Point-in-time recovery

### Backup Strategies
- Regular scheduled backups
- Incremental backups
- Off-site backup storage
- Backup testing and validation

---

## 11. Scaling MongoDB

### Replicasets
- High availability
- Data redundancy
- Automatic failover
- Primary and secondary nodes
- Read preferences
- Replica set configuration

### Sharded Clusters
- Horizontal scaling
- Shard key selection
- Chunk distribution
- Mongos routers
- Config servers
- Sharding strategies (range, hash, zone)

### Tuning Configuration
- Connection pooling
- WiredTiger cache size
- Journal configuration
- Operating system tuning

### Performance Tuning
- **Indexing**: Proper index strategies
- **Query Optimization**: Efficient queries
- Hardware considerations
- Network optimization

---

## 12. MongoDB Security

### Authentication

#### Role-based Access Control (RBAC)
- Built-in roles
- Custom roles
- User management
- Principle of least privilege

#### Authentication Methods
- **X.509 Certificate Auth**: Certificate-based authentication
- **Kerberos Authentication**: Enterprise authentication
- **LDAP Proxy Auth**: LDAP integration
- SCRAM authentication

### MongoDB Audit
- Audit logging
- Compliance requirements
- Tracking database operations
- Security monitoring

### Encryption

#### Encryption at Rest
- Storage-level encryption
- Key management
- Encrypted storage engine

#### Encryption in Transit
- **TLS/SSL Encryption**: Network encryption
- Certificate management
- Secure connections

#### Client-Side Encryption
- **Client-Side Field Level Encryption (CSFLE)**: Encrypt specific fields
- **Queryable Encryption**: Search encrypted data
- Key management systems
- Application-level encryption

---

## Additional Resources

- [MongoDB Official Documentation](https://docs.mongodb.com/)
- [MongoDB University](https://university.mongodb.com/)
- [MongoDB Blog](https://www.mongodb.com/blog)
- [MongoDB Community Forums](https://www.mongodb.com/community/forums/)

---

## Learning Path

1. **Beginner**: Basics, CRUD operations, data modeling
2. **Intermediate**: Indexes, aggregation, replication
3. **Advanced**: Sharding, security, performance tuning
4. **Expert**: Architecture design, scaling strategies, advanced security
