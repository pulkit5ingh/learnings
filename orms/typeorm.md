# TypeORM Interview Questions & Answers

## 1. What is TypeORM?

**Answer:** TypeORM is an Object-Relational Mapping (ORM) library for TypeScript and JavaScript (ES7+) that runs in Node.js, Browser, Ionic, React Native, and Electron. It supports MySQL, PostgreSQL, MariaDB, SQLite, MS SQL Server, Oracle, SAP Hana, and WebSQL.

**Key Features:**
- Works with TypeScript and JavaScript
- Supports Active Record and Data Mapper patterns
- Entity and Column decorators
- Database-agnostic
- Migrations and schema synchronization
- Relations (OneToOne, OneToMany, ManyToMany)
- Eager and lazy loading
- Query Builder
- Connection pooling

```typescript
import { DataSource } from "typeorm";

const AppDataSource = new DataSource({
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: "test",
  password: "test",
  database: "test",
  entities: [User],
  synchronize: true
});
```

---

## 2. What is the difference between Active Record and Data Mapper patterns?

**Answer:**

**Active Record Pattern:**
```typescript
import { BaseEntity, Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User extends BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  // Methods are in the model
  static findByName(name: string) {
    return this.createQueryBuilder("user")
      .where("user.name = :name", { name })
      .getMany();
  }
}

// Usage
const user = new User();
user.name = "John";
await user.save();

const users = await User.find();
const john = await User.findByName("John");
```

**Data Mapper Pattern:**
```typescript
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}

// Separate repository
const userRepository = AppDataSource.getRepository(User);
const user = new User();
user.name = "John";
await userRepository.save(user);

const users = await userRepository.find();
```

| Active Record | Data Mapper |
|---------------|-------------|
| Model has methods | Repository has methods |
| Simpler for small apps | Better for large apps |
| Less separation | Better separation of concerns |
| Extends BaseEntity | Plain classes |

---

## 3. How do you define an Entity in TypeORM?

**Answer:**

```typescript
import {
  Entity,
  PrimaryGeneratedColumn,
  Column,
  CreateDateColumn,
  UpdateDateColumn,
  VersionColumn,
  Index,
  Unique
} from "typeorm";

@Entity({ name: "users" }) // Custom table name
@Index(["email"]) // Simple index
@Index(["firstName", "lastName"]) // Composite index
@Unique(["email"]) // Unique constraint
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ type: "varchar", length: 100 })
  firstName: string;

  @Column()
  lastName: string;

  @Column({ unique: true })
  email: string;

  @Column({ nullable: true })
  age: number;

  @Column({ default: true })
  isActive: boolean;

  @Column("simple-array")
  hobbies: string[];

  @Column("json")
  metadata: object;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @VersionColumn()
  version: number;
}
```

---

## 4. What are the different column types in TypeORM?

**Answer:**

```typescript
@Entity()
export class Example {
  // Numeric types
  @Column("int")
  intColumn: number;

  @Column("bigint")
  bigintColumn: number;

  @Column("float")
  floatColumn: number;

  @Column("decimal", { precision: 10, scale: 2 })
  decimalColumn: number;

  // String types
  @Column("varchar", { length: 255 })
  varcharColumn: string;

  @Column("text")
  textColumn: string;

  @Column("char", { length: 10 })
  charColumn: string;

  // Boolean
  @Column("boolean")
  booleanColumn: boolean;

  // Date and time
  @Column("date")
  dateColumn: Date;

  @Column("time")
  timeColumn: Date;

  @Column("datetime")
  datetimeColumn: Date;

  @Column("timestamp")
  timestampColumn: Date;

  // JSON
  @Column("json")
  jsonColumn: object;

  @Column("jsonb") // PostgreSQL
  jsonbColumn: object;

  // Arrays
  @Column("simple-array")
  simpleArray: string[];

  @Column("simple-json")
  simpleJson: { name: string; age: number };

  // Binary
  @Column("blob")
  blobColumn: Buffer;

  // Enum
  @Column({
    type: "enum",
    enum: ["admin", "user", "guest"],
    default: "user"
  })
  role: string;
}
```

---

## 5. How do you establish relationships in TypeORM?

**Answer:**

**One-to-One:**
```typescript
import { Entity, PrimaryGeneratedColumn, Column, OneToOne, JoinColumn } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToOne(() => Profile, profile => profile.user, { cascade: true })
  @JoinColumn()
  profile: Profile;
}

@Entity()
export class Profile {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  bio: string;

  @OneToOne(() => User, user => user.profile)
  user: User;
}
```

**One-to-Many / Many-to-One:**
```typescript
@Entity()
export class Author {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author)
  posts: Post[];
}

@Entity()
export class Post {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @ManyToOne(() => Author, author => author.posts, { onDelete: "CASCADE" })
  author: Author;
}
```

**Many-to-Many:**
```typescript
@Entity()
export class Student {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Course, course => course.students)
  @JoinTable()
  courses: Course[];
}

@Entity()
export class Course {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @ManyToMany(() => Student, student => student.courses)
  students: Student[];
}
```

---

## 6. How do you perform CRUD operations?

**Answer:**

```typescript
const userRepository = AppDataSource.getRepository(User);

// CREATE
const user = new User();
user.name = "John";
user.email = "john@example.com";
await userRepository.save(user);

// Or using create
const user = userRepository.create({
  name: "John",
  email: "john@example.com"
});
await userRepository.save(user);

// Or using insert
await userRepository.insert({
  name: "John",
  email: "john@example.com"
});

// READ
const users = await userRepository.find();
const user = await userRepository.findOne({ where: { id: 1 } });
const user = await userRepository.findOneBy({ email: "john@example.com" });

// UPDATE
await userRepository.update({ id: 1 }, { name: "John Doe" });

// Or
const user = await userRepository.findOneBy({ id: 1 });
user.name = "John Doe";
await userRepository.save(user);

// DELETE
await userRepository.delete({ id: 1 });
await userRepository.remove(user);

// Soft delete
await userRepository.softDelete({ id: 1 });

// Restore soft deleted
await userRepository.restore({ id: 1 });
```

---

## 7. What is Query Builder?

**Answer:** Query Builder provides a more flexible way to build SQL queries.

```typescript
// Basic query
const users = await userRepository
  .createQueryBuilder("user")
  .where("user.age > :age", { age: 18 })
  .getMany();

// Complex query
const users = await userRepository
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.profile", "profile")
  .where("user.isActive = :isActive", { isActive: true })
  .andWhere("user.age BETWEEN :min AND :max", { min: 18, max: 65 })
  .orderBy("user.createdAt", "DESC")
  .take(10)
  .skip(0)
  .getMany();

// Joins
const users = await userRepository
  .createQueryBuilder("user")
  .innerJoinAndSelect("user.posts", "post")
  .leftJoinAndSelect("user.profile", "profile")
  .getMany();

// Aggregation
const result = await userRepository
  .createQueryBuilder("user")
  .select("user.role", "role")
  .addSelect("COUNT(user.id)", "count")
  .addSelect("AVG(user.age)", "avgAge")
  .groupBy("user.role")
  .getRawMany();

// Subquery
const posts = await postRepository
  .createQueryBuilder("post")
  .where(qb => {
    const subQuery = qb
      .subQuery()
      .select("user.id")
      .from(User, "user")
      .where("user.isActive = :isActive", { isActive: true })
      .getQuery();
    return "post.authorId IN " + subQuery;
  })
  .getMany();

// Insert
await userRepository
  .createQueryBuilder()
  .insert()
  .into(User)
  .values([
    { name: "John", email: "john@example.com" },
    { name: "Jane", email: "jane@example.com" }
  ])
  .execute();

// Update
await userRepository
  .createQueryBuilder()
  .update(User)
  .set({ name: "John Doe" })
  .where("id = :id", { id: 1 })
  .execute();

// Delete
await userRepository
  .createQueryBuilder()
  .delete()
  .from(User)
  .where("id = :id", { id: 1 })
  .execute();
```

---

## 8. How do you handle transactions?

**Answer:**

```typescript
// Method 1: Using queryRunner
const queryRunner = AppDataSource.createQueryRunner();
await queryRunner.connect();
await queryRunner.startTransaction();

try {
  const user = new User();
  user.name = "John";
  await queryRunner.manager.save(user);

  const profile = new Profile();
  profile.user = user;
  await queryRunner.manager.save(profile);

  await queryRunner.commitTransaction();
} catch (err) {
  await queryRunner.rollbackTransaction();
  throw err;
} finally {
  await queryRunner.release();
}

// Method 2: Using transaction wrapper
await AppDataSource.transaction(async manager => {
  const user = new User();
  user.name = "John";
  await manager.save(user);

  const profile = new Profile();
  profile.user = user;
  await manager.save(profile);
});

// Method 3: Isolation levels
await AppDataSource.transaction("SERIALIZABLE", async manager => {
  // Transaction code
});

// Available isolation levels:
// READ UNCOMMITTED
// READ COMMITTED
// REPEATABLE READ
// SERIALIZABLE
```

---

## 9. What are Entity Listeners and Subscribers?

**Answer:**

**Entity Listeners:**
```typescript
import { Entity, PrimaryGeneratedColumn, Column, BeforeInsert, AfterInsert, BeforeUpdate, AfterUpdate, BeforeRemove, AfterRemove } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @BeforeInsert()
  beforeInsertActions() {
    console.log("Before insert");
    this.name = this.name.toUpperCase();
  }

  @AfterInsert()
  afterInsertActions() {
    console.log("After insert:", this.id);
  }

  @BeforeUpdate()
  beforeUpdateActions() {
    console.log("Before update");
  }

  @AfterUpdate()
  afterUpdateActions() {
    console.log("After update");
  }

  @BeforeRemove()
  beforeRemoveActions() {
    console.log("Before remove");
  }

  @AfterRemove()
  afterRemoveActions() {
    console.log("After remove");
  }
}
```

**Subscribers:**
```typescript
import { EventSubscriber, EntitySubscriberInterface, InsertEvent, UpdateEvent } from "typeorm";

@EventSubscriber()
export class UserSubscriber implements EntitySubscriberInterface<User> {
  listenTo() {
    return User;
  }

  beforeInsert(event: InsertEvent<User>) {
    console.log("BEFORE USER INSERTED:", event.entity);
  }

  afterInsert(event: InsertEvent<User>) {
    console.log("AFTER USER INSERTED:", event.entity);
  }

  beforeUpdate(event: UpdateEvent<User>) {
    console.log("BEFORE USER UPDATED:", event.entity);
  }

  afterUpdate(event: UpdateEvent<User>) {
    console.log("AFTER USER UPDATED:", event.entity);
  }
}

// Register subscriber
const AppDataSource = new DataSource({
  // ...
  subscribers: [UserSubscriber]
});
```

---

## 10. How do you implement migrations?

**Answer:**

```typescript
// typeorm.config.ts
export default new DataSource({
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: "test",
  password: "test",
  database: "test",
  entities: ["src/entity/**/*.ts"],
  migrations: ["src/migration/**/*.ts"],
  synchronize: false
});

// Generate migration
// npx typeorm migration:generate -n CreateUserTable

// Migration file: src/migration/1234567890123-CreateUserTable.ts
import { MigrationInterface, QueryRunner, Table } from "typeorm";

export class CreateUserTable1234567890123 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: "users",
        columns: [
          {
            name: "id",
            type: "int",
            isPrimary: true,
            isGenerated: true,
            generationStrategy: "increment"
          },
          {
            name: "name",
            type: "varchar",
            length: "100"
          },
          {
            name: "email",
            type: "varchar",
            isUnique: true
          },
          {
            name: "created_at",
            type: "timestamp",
            default: "CURRENT_TIMESTAMP"
          }
        ]
      })
    );

    // Create index
    await queryRunner.createIndex("users",
      new TableIndex({
        name: "IDX_USER_EMAIL",
        columnNames: ["email"]
      })
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable("users");
  }
}

// Run migrations
// npx typeorm migration:run

// Revert migration
// npx typeorm migration:revert
```

---

## 11. What are Embedded Entities?

**Answer:**

```typescript
import { Entity, Column } from "typeorm";

// Embedded entity
export class Name {
  @Column()
  first: string;

  @Column()
  last: string;
}

// Use embedded entity
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column(() => Name)
  name: Name;

  @Column()
  email: string;
}

// Usage
const user = new User();
user.name = new Name();
user.name.first = "John";
user.name.last = "Doe";
user.email = "john@example.com";
await userRepository.save(user);

// Database structure:
// users table:
// - id
// - nameFirst
// - nameLast
// - email

// With prefix
@Column(() => Name, { prefix: "contact_" })
contactName: Name;
// Result: contact_first, contact_last
```

---

## 12. How do you implement pagination?

**Answer:**

```typescript
// Method 1: Using skip and take
async function paginate(page: number, limit: number) {
  const skip = (page - 1) * limit;

  const [data, total] = await userRepository.findAndCount({
    skip,
    take: limit,
    order: { createdAt: "DESC" }
  });

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

// Method 2: Using Query Builder
async function paginateWithBuilder(page: number, limit: number) {
  const skip = (page - 1) * limit;

  const [data, total] = await userRepository
    .createQueryBuilder("user")
    .skip(skip)
    .take(limit)
    .orderBy("user.createdAt", "DESC")
    .getManyAndCount();

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

// Method 3: Cursor-based pagination
async function cursorPaginate(cursor: number | null, limit: number) {
  const qb = userRepository
    .createQueryBuilder("user")
    .take(limit + 1)
    .orderBy("user.id", "ASC");

  if (cursor) {
    qb.where("user.id > :cursor", { cursor });
  }

  const data = await qb.getMany();
  const hasMore = data.length > limit;
  const results = hasMore ? data.slice(0, -1) : data;
  const nextCursor = hasMore ? results[results.length - 1].id : null;

  return {
    data: results,
    nextCursor,
    hasMore
  };
}
```

---

## 13. What is Eager and Lazy Loading?

**Answer:**

**Eager Loading:**
```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author, { eager: true })
  posts: Post[];
}

// Posts are automatically loaded
const user = await userRepository.findOne({ where: { id: 1 } });
console.log(user.posts); // Available without explicit join
```

**Lazy Loading:**
```typescript
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @OneToMany(() => Post, post => post.author)
  posts: Promise<Post[]>;
}

// Posts loaded on access
const user = await userRepository.findOne({ where: { id: 1 } });
const posts = await user.posts; // Lazy loaded
```

**Manual Loading (Recommended):**
```typescript
// Using relations option
const user = await userRepository.findOne({
  where: { id: 1 },
  relations: ["posts"]
});

// Using Query Builder
const user = await userRepository
  .createQueryBuilder("user")
  .leftJoinAndSelect("user.posts", "posts")
  .where("user.id = :id", { id: 1 })
  .getOne();
```

---

## 14. How do you implement custom repositories?

**Answer:**

```typescript
// Custom repository using Data Mapper
@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;
}

export class UserRepository extends Repository<User> {
  async findByEmail(email: string): Promise<User | null> {
    return this.findOne({ where: { email } });
  }

  async findActiveUsers(): Promise<User[]> {
    return this.createQueryBuilder("user")
      .where("user.isActive = :isActive", { isActive: true })
      .getMany();
  }

  async searchByName(name: string): Promise<User[]> {
    return this.createQueryBuilder("user")
      .where("user.name LIKE :name", { name: `%${name}%` })
      .getMany();
  }
}

// Register custom repository
const AppDataSource = new DataSource({
  // ...
  entities: [User],
});

// Get custom repository
const userRepository = AppDataSource.getRepository(User).extend({
  async findByEmail(email: string) {
    return this.findOne({ where: { email } });
  },

  async findActiveUsers() {
    return this.find({ where: { isActive: true } });
  }
});

// Usage
const user = await userRepository.findByEmail("john@example.com");
const activeUsers = await userRepository.findActiveUsers();
```

---

## 15. What are indexes and how do you create them?

**Answer:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from "typeorm";

@Entity()
// Single column index
@Index("IDX_USER_EMAIL", ["email"])
// Composite index
@Index("IDX_USER_NAME", ["firstName", "lastName"])
// Unique index
@Index("IDX_USER_USERNAME", ["username"], { unique: true })
// Partial index (PostgreSQL)
@Index("IDX_ACTIVE_USERS", ["email"], { where: '"isActive" = true' })
// Full-text index (MySQL)
@Index("IDX_USER_FULLTEXT", ["name", "bio"], { fulltext: true })
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  @Index() // Simple index on single column
  email: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ unique: true }) // Unique constraint
  username: string;

  @Column()
  bio: string;

  @Column({ default: true })
  isActive: boolean;
}

// Spatial index
@Entity()
@Index("IDX_LOCATION", ["location"], { spatial: true })
export class Place {
  @PrimaryGeneratedColumn()
  id: number;

  @Column("geometry")
  location: string;
}

// Create index in migration
await queryRunner.createIndex("users",
  new TableIndex({
    name: "IDX_USER_CREATED",
    columnNames: ["createdAt"]
  })
);
```

---

## 16. How do you handle soft deletes?

**Answer:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column, DeleteDateColumn } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column()
  email: string;

  @DeleteDateColumn()
  deletedAt: Date;
}

// Soft delete
await userRepository.softDelete({ id: 1 });

// Or
const user = await userRepository.findOne({ where: { id: 1 } });
await userRepository.softRemove(user);

// Restore
await userRepository.restore({ id: 1 });

// Or
const user = await userRepository.findOne({
  where: { id: 1 },
  withDeleted: true
});
await userRepository.recover(user);

// Find with deleted
const users = await userRepository.find({ withDeleted: true });

// Find only deleted
const deletedUsers = await userRepository
  .createQueryBuilder("user")
  .where("user.deletedAt IS NOT NULL")
  .withDeleted()
  .getMany();

// Permanent delete
await userRepository.delete({ id: 1 });
```

---

## 17. What are View Entities?

**Answer:**

```typescript
import { ViewEntity, ViewColumn, Connection } from "typeorm";

// Database view
@ViewEntity({
  expression: (connection: Connection) =>
    connection
      .createQueryBuilder()
      .select("user.id", "id")
      .addSelect("user.name", "name")
      .addSelect("COUNT(post.id)", "postCount")
      .from(User, "user")
      .leftJoin("user.posts", "post")
      .groupBy("user.id")
})
export class UserStats {
  @ViewColumn()
  id: number;

  @ViewColumn()
  name: string;

  @ViewColumn()
  postCount: number;
}

// Query view
const stats = await connection.getRepository(UserStats).find();

// Raw SQL view
@ViewEntity({
  name: "user_stats",
  expression: `
    SELECT
      u.id,
      u.name,
      COUNT(p.id) as postCount
    FROM users u
    LEFT JOIN posts p ON p.authorId = u.id
    GROUP BY u.id
  `
})
export class UserStats {
  @ViewColumn()
  id: number;

  @ViewColumn()
  name: string;

  @ViewColumn()
  postCount: number;
}
```

---

## 18. How do you implement inheritance?

**Answer:**

**Single Table Inheritance:**
```typescript
@Entity()
@TableInheritance({ column: { type: "varchar", name: "type" } })
export class Content {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;
}

@ChildEntity()
export class Post extends Content {
  @Column()
  content: string;
}

@ChildEntity()
export class Video extends Content {
  @Column()
  url: string;

  @Column()
  duration: number;
}

// All stored in single 'content' table with 'type' discriminator
```

**Concrete Table Inheritance:**
```typescript
export abstract class Content {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;
}

@Entity()
export class Post extends Content {
  @Column()
  content: string;
}

@Entity()
export class Video extends Content {
  @Column()
  url: string;
}

// Each entity has its own table
```

---

## 19. How do you use raw SQL queries?

**Answer:**

```typescript
// Using query()
const users = await AppDataSource.query(
  "SELECT * FROM users WHERE age > $1",
  [18]
);

// Using Query Builder raw
const users = await userRepository
  .createQueryBuilder("user")
  .where("user.age > :age", { age: 18 })
  .andWhere("DATE_PART('year', AGE(user.birthDate)) >= :minAge", { minAge: 18 })
  .getRawMany();

// Raw SQL with select
const result = await userRepository
  .createQueryBuilder("user")
  .select("COUNT(*)", "count")
  .addSelect("user.role", "role")
  .groupBy("user.role")
  .getRawMany();

// Execute raw query
await AppDataSource.manager.query(
  `UPDATE users SET status = $1 WHERE id = $2`,
  ["active", userId]
);

// With entity manager
const rawData = await AppDataSource.manager
  .createQueryBuilder()
  .select("user")
  .from(User, "user")
  .where("user.id = :id", { id: 1 })
  .getRawOne();

// Named parameters
const users = await AppDataSource.query(
  "SELECT * FROM users WHERE email = :email AND age > :age",
  { email: "john@example.com", age: 18 }
);
```

---

## 20. How do you handle connection pooling?

**Answer:**

```typescript
const AppDataSource = new DataSource({
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: "test",
  password: "test",
  database: "test",
  entities: [User],

  // Connection pool settings
  extra: {
    // Maximum number of clients in the pool
    max: 10,

    // Minimum number of clients
    min: 2,

    // Maximum time to wait for a connection (ms)
    connectionTimeoutMillis: 2000,

    // Maximum time a client can remain idle (ms)
    idleTimeoutMillis: 30000,

    // How often to check for idle clients (ms)
    reapIntervalMillis: 1000,

    // Function called when acquiring connection
    onConnect: (connection) => {
      console.log("New connection");
    }
  },

  // Connection retry
  maxQueryExecutionTime: 1000,

  // Additional options
  poolSize: 10, // Deprecated, use extra.max

  // Logging
  logging: ["error", "warn"],
  logger: "advanced-console"
});

// Monitor pool
AppDataSource.driver.pool?.on("acquire", (connection) => {
  console.log("Connection acquired");
});

AppDataSource.driver.pool?.on("release", (connection) => {
  console.log("Connection released");
});
```

---

## 21. How do you implement Tree Entities?

**Answer:**

**Adjacency List:**
```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, OneToMany, Tree, TreeChildren, TreeParent } from "typeorm";

@Entity()
@Tree("adjacency-list")
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeChildren()
  children: Category[];

  @TreeParent()
  parent: Category;
}

// Usage
const categoryRepository = AppDataSource.getTreeRepository(Category);

// Find trees
const trees = await categoryRepository.findTrees();

// Find descendants
const descendants = await categoryRepository.findDescendants(category);

// Find ancestors
const ancestors = await categoryRepository.findAncestors(category);

// Count descendants
const count = await categoryRepository.countDescendants(category);
```

**Nested Set:**
```typescript
@Entity()
@Tree("nested-set")
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeChildren()
  children: Category[];

  @TreeParent()
  parent: Category;
}
```

**Closure Table:**
```typescript
@Entity()
@Tree("closure-table")
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeChildren()
  children: Category[];

  @TreeParent()
  parent: Category;
}
```

**Materialized Path:**
```typescript
@Entity()
@Tree("materialized-path")
export class Category {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @TreeChildren()
  children: Category[];

  @TreeParent()
  parent: Category;
}
```

---

## 22. How do you implement caching?

**Answer:**

```typescript
// Enable caching
const AppDataSource = new DataSource({
  type: "postgres",
  // ...
  cache: true,

  // Or with Redis
  cache: {
    type: "redis",
    options: {
      host: "localhost",
      port: 6379
    },
    duration: 30000 // 30 seconds
  },

  // Or with ioredis
  cache: {
    type: "ioredis",
    options: {
      host: "localhost",
      port: 6379
    }
  }
});

// Cache query
const users = await userRepository.find({
  cache: true
});

// Cache with custom TTL
const users = await userRepository.find({
  cache: 60000 // 60 seconds
});

// Cache with key
const users = await userRepository.find({
  cache: {
    id: "users_list",
    milliseconds: 60000
  }
});

// Query Builder caching
const users = await userRepository
  .createQueryBuilder("user")
  .where("user.isActive = :isActive", { isActive: true })
  .cache(true)
  .getMany();

// With custom cache key
const users = await userRepository
  .createQueryBuilder("user")
  .cache("active_users", 60000)
  .getMany();

// Clear cache
await AppDataSource.queryResultCache?.clear();

// Remove specific cache
await AppDataSource.queryResultCache?.remove(["users_list"]);

// Get cache statistics
const count = await AppDataSource.queryResultCache?.getCount();
```

---

## 23. What are Entity Schemas (without decorators)?

**Answer:**

```typescript
import { EntitySchema } from "typeorm";

// Define entity using EntitySchema
export const UserSchema = new EntitySchema({
  name: "User",
  tableName: "users",
  columns: {
    id: {
      type: Number,
      primary: true,
      generated: true
    },
    name: {
      type: String,
      length: 100
    },
    email: {
      type: String,
      unique: true
    },
    age: {
      type: Number,
      nullable: true
    },
    isActive: {
      type: Boolean,
      default: true
    },
    createdAt: {
      type: Date,
      createDate: true
    },
    updatedAt: {
      type: Date,
      updateDate: true
    }
  },
  relations: {
    posts: {
      type: "one-to-many",
      target: "Post",
      inverseSide: "author"
    },
    profile: {
      type: "one-to-one",
      target: "Profile",
      joinColumn: true,
      cascade: true
    }
  },
  indices: [
    {
      name: "IDX_USER_EMAIL",
      columns: ["email"]
    }
  ]
});

// Use with DataSource
const AppDataSource = new DataSource({
  type: "postgres",
  entities: [UserSchema, PostSchema]
});
```

---

## 24. How do you handle database-specific features?

**Answer:**

```typescript
// PostgreSQL specific
@Entity()
export class User {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column("jsonb")
  metadata: object;

  @Column("array")
  tags: string[];

  @Index({ fulltext: true })
  @Column("text")
  content: string;

  // PostgreSQL full-text search
  static async searchByText(searchTerm: string) {
    return this.createQueryBuilder("user")
      .where("to_tsvector(user.content) @@ to_tsquery(:search)", {
        search: searchTerm
      })
      .getMany();
  }
}

// MySQL specific
@Entity()
export class Product {
  @Index({ fulltext: true, parser: "ngram" })
  @Column("text")
  description: string;

  // MySQL JSON operations
  static async findByJsonPath() {
    return this.createQueryBuilder("product")
      .where("JSON_EXTRACT(metadata, '$.category') = :category", {
        category: "electronics"
      })
      .getMany();
  }
}

// MongoDB specific (using TypeORM with MongoDB)
@Entity()
export class Log {
  @ObjectIdColumn()
  id: ObjectId;

  @Column()
  message: string;

  @CreateDateColumn()
  createdAt: Date;
}

// SQL Server specific
@Entity()
export class Document {
  @PrimaryGeneratedColumn()
  id: number;

  @Column("nvarchar", { length: "MAX" })
  content: string;

  // Use SQL Server specific types
  @Column("hierarchyid")
  hierarchy: string;
}
```

---

## 25. How do you implement database seeding?

**Answer:**

```typescript
// seeds/UserSeeder.ts
import { DataSource } from "typeorm";
import { User } from "../entity/User";

export class UserSeeder {
  async run(dataSource: DataSource): Promise<void> {
    const userRepository = dataSource.getRepository(User);

    // Clear existing data
    await userRepository.clear();

    // Create users
    const users = [
      {
        name: "John Doe",
        email: "john@example.com",
        age: 30
      },
      {
        name: "Jane Smith",
        email: "jane@example.com",
        age: 25
      }
    ];

    await userRepository.save(users);
    console.log("Users seeded");
  }
}

// Run seeder
import { AppDataSource } from "./data-source";
import { UserSeeder } from "./seeds/UserSeeder";

AppDataSource.initialize()
  .then(async (dataSource) => {
    const seeder = new UserSeeder();
    await seeder.run(dataSource);
    await dataSource.destroy();
  })
  .catch((error) => console.log(error));

// Using Faker for random data
import { faker } from "@faker-js/faker";

export class UserSeeder {
  async run(dataSource: DataSource): Promise<void> {
    const userRepository = dataSource.getRepository(User);

    const users = Array.from({ length: 100 }, () => ({
      name: faker.person.fullName(),
      email: faker.internet.email(),
      age: faker.number.int({ min: 18, max: 80 })
    }));

    await userRepository.save(users);
  }
}
```

---

## 26. How do you handle multiple databases?

**Answer:**

```typescript
// Define multiple data sources
export const PostgresDataSource = new DataSource({
  name: "postgres",
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: "test",
  password: "test",
  database: "testdb",
  entities: [User, Post]
});

export const MySQLDataSource = new DataSource({
  name: "mysql",
  type: "mysql",
  host: "localhost",
  port: 3306,
  username: "test",
  password: "test",
  database: "testdb",
  entities: [Product, Order]
});

// Initialize connections
await PostgresDataSource.initialize();
await MySQLDataSource.initialize();

// Use different databases
const userRepository = PostgresDataSource.getRepository(User);
const productRepository = MySQLDataSource.getRepository(Product);

const users = await userRepository.find();
const products = await productRepository.find();

// Entity with specific connection
@Entity({ database: "secondary" })
export class Log {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  message: string;
}
```

---

## 27. How do you implement optimistic locking?

**Answer:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column, VersionColumn } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @VersionColumn()
  version: number;
}

// Usage
try {
  const user = await userRepository.findOne({ where: { id: 1 } });
  user.name = "John Doe";
  await userRepository.save(user);
} catch (error) {
  if (error.name === "OptimisticLockVersionMismatchError") {
    console.log("Version conflict - document was modified by another transaction");
    // Handle conflict
  }
}

// Manual version checking
const user = await userRepository.findOne({ where: { id: 1 } });
const currentVersion = user.version;

user.name = "John Doe";
const result = await userRepository.update(
  { id: 1, version: currentVersion },
  { name: "John Doe", version: currentVersion + 1 }
);

if (result.affected === 0) {
  throw new Error("Version conflict");
}
```

---

## 28. What are Generated Columns?

**Answer:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Generated } from "typeorm";

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  @Generated("uuid")
  uuid: string;

  @Column()
  @Generated("increment")
  version: number;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  // Virtual generated column (PostgreSQL, MySQL 5.7+)
  @Column({
    type: "varchar",
    generatedType: "VIRTUAL",
    asExpression: "CONCAT(firstName, ' ', lastName)"
  })
  fullName: string;

  // Stored generated column
  @Column({
    type: "int",
    generatedType: "STORED",
    asExpression: "YEAR(CURRENT_DATE) - birthYear"
  })
  age: number;

  @Column()
  birthYear: number;
}

// Database will automatically generate values
const user = new User();
user.firstName = "John";
user.lastName = "Doe";
user.birthYear = 1990;
await userRepository.save(user);
// fullName and age are automatically calculated
```

---

## 29. How do you implement database logging and monitoring?

**Answer:**

```typescript
import { Logger } from "typeorm";

// Custom logger
class MyCustomLogger implements Logger {
  logQuery(query: string, parameters?: any[]) {
    console.log(`Query: ${query}`);
    console.log(`Parameters: ${JSON.stringify(parameters)}`);
  }

  logQueryError(error: string, query: string, parameters?: any[]) {
    console.error(`Query Error: ${error}`);
    console.error(`Failed Query: ${query}`);
  }

  logQuerySlow(time: number, query: string, parameters?: any[]) {
    console.warn(`Slow Query (${time}ms): ${query}`);
  }

  logSchemaBuild(message: string) {
    console.log(`Schema: ${message}`);
  }

  logMigration(message: string) {
    console.log(`Migration: ${message}`);
  }

  log(level: "log" | "info" | "warn", message: any) {
    console[level](message);
  }
}

// Use custom logger
const AppDataSource = new DataSource({
  type: "postgres",
  // ...
  logger: new MyCustomLogger(),
  logging: ["query", "error", "schema", "warn", "info", "log"],
  maxQueryExecutionTime: 1000 // Log slow queries
});

// Built-in loggers
// "advanced-console" - detailed logging
// "simple-console" - simple logging
// "file" - log to file
// "debug" - debug level logging

// Monitor query performance
AppDataSource.driver.connection?.on("query", (query) => {
  console.log("Query executed:", query);
});
```

---

## 30. How do you implement database-level constraints?

**Answer:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Unique, Check } from "typeorm";

@Entity()
@Unique(["email"])
@Unique(["username", "tenantId"])
@Check(`"age" >= 18`)
@Check(`"salary" > 0`)
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  username: string;

  @Column()
  email: string;

  @Column()
  age: number;

  @Column("decimal")
  salary: number;

  @Column()
  tenantId: number;
}

// In migration
public async up(queryRunner: QueryRunner): Promise<void> {
  // Add unique constraint
  await queryRunner.createUniqueConstraint("users",
    new TableUnique({
      name: "UQ_USER_EMAIL",
      columnNames: ["email"]
    })
  );

  // Add check constraint
  await queryRunner.createCheckConstraint("users",
    new TableCheck({
      name: "CHK_USER_AGE",
      expression: `"age" >= 18`
    })
  );

  // Add foreign key constraint
  await queryRunner.createForeignKey("posts",
    new TableForeignKey({
      name: "FK_POST_AUTHOR",
      columnNames: ["authorId"],
      referencedTableName: "users",
      referencedColumnNames: ["id"],
      onDelete: "CASCADE",
      onUpdate: "CASCADE"
    })
  );

  // Add exclusion constraint (PostgreSQL)
  await queryRunner.query(`
    ALTER TABLE bookings
    ADD CONSTRAINT no_overlapping_bookings
    EXCLUDE USING gist (room WITH =, daterange WITH &&)
  `);
}
```

---

## 31. How do you handle spatial data?

**Answer:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column, Index } from "typeorm";

@Entity()
@Index("IDX_LOCATION", ["location"], { spatial: true })
export class Place {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;

  @Column({
    type: "geography",
    spatialFeatureType: "Point",
    srid: 4326
  })
  location: string;

  // For MySQL
  @Column({
    type: "point",
    nullable: true
  })
  coordinates: string;
}

// Store location (PostgreSQL with PostGIS)
const place = new Place();
place.name = "Times Square";
place.location = "POINT(-73.985130 40.758896)";
await placeRepository.save(place);

// Find places near a point
const places = await placeRepository
  .createQueryBuilder("place")
  .where("ST_DWithin(place.location, ST_MakePoint(:lng, :lat)::geography, :distance)", {
    lng: -73.985130,
    lat: 40.758896,
    distance: 5000 // 5km in meters
  })
  .getMany();

// Find places within radius
const places = await placeRepository
  .createQueryBuilder("place")
  .where(
    "ST_Distance(place.location, ST_MakePoint(:lng, :lat)::geography) <= :radius",
    {
      lng: -73.985130,
      lat: 40.758896,
      radius: 5000
    }
  )
  .getMany();

// MySQL spatial queries
const places = await placeRepository
  .createQueryBuilder("place")
  .where(
    "ST_Distance_Sphere(place.coordinates, POINT(:lng, :lat)) <= :distance",
    {
      lng: -73.985130,
      lat: 40.758896,
      distance: 5000
    }
  )
  .getMany();
```

---

## 32. How do you implement database replication?

**Answer:**

```typescript
// Master-slave replication setup
const AppDataSource = new DataSource({
  type: "mysql",
  replication: {
    master: {
      host: "master.example.com",
      port: 3306,
      username: "root",
      password: "password",
      database: "test"
    },
    slaves: [
      {
        host: "slave1.example.com",
        port: 3306,
        username: "root",
        password: "password",
        database: "test"
      },
      {
        host: "slave2.example.com",
        port: 3306,
        username: "root",
        password: "password",
        database: "test"
      }
    ],

    // Slave selection strategy
    // "RR" - round-robin (default)
    // "RANDOM" - random selection
    // "ORDER" - ordered selection
    selector: "RR",

    // Remove failed slaves
    removeNodeErrorCount: 5,

    // Restore failed slaves
    restoreNodeTimeout: 0
  },
  entities: [User]
});

// Writes go to master, reads to slaves automatically
const user = new User();
user.name = "John";
await userRepository.save(user); // Writes to master

const users = await userRepository.find(); // Reads from slave

// Force read from master
const users = await userRepository
  .createQueryBuilder("user")
  .setQueryRunner(AppDataSource.createQueryRunner("master"))
  .getMany();
```

---

## 33. How do you implement data transformation?

**Answer:**

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ValueTransformer } from "typeorm";

// Custom transformer
const encryptTransformer: ValueTransformer = {
  to(value: string): string {
    // Encrypt before saving to database
    return encrypt(value);
  },
  from(value: string): string {
    // Decrypt when reading from database
    return decrypt(value);
  }
};

// JSON transformer
const jsonTransformer: ValueTransformer = {
  to(value: any): string {
    return JSON.stringify(value);
  },
  from(value: string): any {
    return JSON.parse(value);
  }
};

@Entity()
export class User {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({
    type: "varchar",
    transformer: encryptTransformer
  })
  password: string;

  @Column({
    type: "text",
    transformer: jsonTransformer
  })
  settings: object;

  // Date transformer
  @Column({
    type: "varchar",
    transformer: {
      to(value: Date): string {
        return value.toISOString();
      },
      from(value: string): Date {
        return new Date(value);
      }
    }
  })
  customDate: Date;

  // Array transformer
  @Column({
    type: "text",
    transformer: {
      to(value: string[]): string {
        return value.join(",");
      },
      from(value: string): string[] {
        return value.split(",");
      }
    }
  })
  tags: string[];
}
```

---

## 34. What are the best practices for TypeORM?

**Answer:**

**1. Use Migrations (not synchronize in production)**
```typescript
synchronize: false, // Never true in production
migrations: ["src/migration/**/*.ts"]
```

**2. Use Query Builder for complex queries**
```typescript
// Instead of find with complex where
const users = await userRepository
  .createQueryBuilder("user")
  .where("user.age > :age", { age: 18 })
  .getMany();
```

**3. Use transactions for related operations**
```typescript
await AppDataSource.transaction(async manager => {
  await manager.save(user);
  await manager.save(profile);
});
```

**4. Index frequently queried columns**
```typescript
@Index()
@Column()
email: string;
```

**5. Use select to limit fields**
```typescript
const users = await userRepository.find({
  select: ["id", "name", "email"]
});
```

**6. Avoid N+1 queries**
```typescript
// Bad
const posts = await postRepository.find();
for (const post of posts) {
  await post.author; // N queries
}

// Good
const posts = await postRepository.find({
  relations: ["author"]
});
```

**7. Use connection pooling**
**8. Handle errors properly**
**9. Use prepared statements (parameterized queries)**
**10. Monitor slow queries**

---

## 35. How do you implement full-text search?

**Answer:**

```typescript
// PostgreSQL full-text search
@Entity()
export class Article {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column("text")
  content: string;

  @Index({ fulltext: true })
  @Column("tsvector", { nullable: true, select: false })
  searchVector: string;
}

// Create search vector trigger
await queryRunner.query(`
  CREATE TRIGGER article_search_vector_update
  BEFORE INSERT OR UPDATE ON articles
  FOR EACH ROW EXECUTE FUNCTION
  tsvector_update_trigger(
    search_vector, 'pg_catalog.english', title, content
  );
`);

// Search
const articles = await articleRepository
  .createQueryBuilder("article")
  .where("article.searchVector @@ to_tsquery('english', :query)", {
    query: "javascript & typescript"
  })
  .orderBy("ts_rank(article.searchVector, to_tsquery('english', :query))", "DESC")
  .getMany();

// MySQL full-text search
@Entity()
@Index("IDX_ARTICLE_FULLTEXT", ["title", "content"], { fulltext: true })
export class Article {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column("text")
  content: string;
}

// Search
const articles = await articleRepository
  .createQueryBuilder("article")
  .where("MATCH(article.title, article.content) AGAINST(:query IN BOOLEAN MODE)", {
    query: "+javascript +typescript"
  })
  .getMany();
```

---

## 36. How do you implement audit trails?

**Answer:**

```typescript
// Base entity with audit fields
export abstract class AuditEntity {
  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @Column({ nullable: true })
  createdBy: number;

  @Column({ nullable: true })
  updatedBy: number;
}

@Entity()
export class User extends AuditEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  name: string;
}

// Audit log entity
@Entity()
export class AuditLog {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  entityName: string;

  @Column()
  entityId: number;

  @Column()
  action: string;

  @Column("json")
  oldValues: any;

  @Column("json")
  newValues: any;

  @Column()
  userId: number;

  @CreateDateColumn()
  timestamp: Date;
}

// Subscriber for audit logging
@EventSubscriber()
export class AuditSubscriber implements EntitySubscriberInterface {
  async afterUpdate(event: UpdateEvent<any>) {
    const auditLog = new AuditLog();
    auditLog.entityName = event.metadata.name;
    auditLog.entityId = event.entity.id;
    auditLog.action = "UPDATE";
    auditLog.oldValues = event.databaseEntity;
    auditLog.newValues = event.entity;
    await event.manager.save(auditLog);
  }
}
```

---

## 37. How do you handle database schema validation?

**Answer:**

```typescript
// Synchronize schema (development only)
const AppDataSource = new DataSource({
  synchronize: true, // Creates tables automatically
  dropSchema: false, // Drop schema on connection
  migrationsRun: false
});

// Check if schema needs sync
await AppDataSource.synchronize(); // Manual sync

// Validate schema
try {
  await AppDataSource.synchronize(false); // Dry run
} catch (error) {
  console.error("Schema out of sync:", error);
}

// Generate migration from current schema
// npx typeorm migration:generate -n SyncSchema

// Schema validation in tests
describe("Database Schema", () => {
  it("should match entity definitions", async () => {
    const queryRunner = AppDataSource.createQueryRunner();
    const table = await queryRunner.getTable("users");

    expect(table).toBeDefined();
    expect(table.columns.find(c => c.name === "email")).toBeDefined();
    expect(table.indices.find(i => i.name === "IDX_USER_EMAIL")).toBeDefined();

    await queryRunner.release();
  });
});
```

---

## 38. How do you implement pessimistic locking?

**Answer:**

```typescript
// Pessimistic read lock
const user = await userRepository.findOne({
  where: { id: 1 },
  lock: { mode: "pessimistic_read" }
});

// Pessimistic write lock
const user = await userRepository.findOne({
  where: { id: 1 },
  lock: { mode: "pessimistic_write" }
});

// With Query Builder
const user = await userRepository
  .createQueryBuilder("user")
  .where("user.id = :id", { id: 1 })
  .setLock("pessimistic_write")
  .getOne();

// In transaction
await AppDataSource.transaction(async manager => {
  const user = await manager.findOne(User, {
    where: { id: 1 },
    lock: { mode: "pessimistic_write" }
  });

  user.balance -= 100;
  await manager.save(user);
});

// Dirty read (PostgreSQL)
const user = await userRepository.findOne({
  where: { id: 1 },
  lock: { mode: "dirty_read" }
});

// Pessimistic partial write lock (MySQL)
await userRepository.findOne({
  where: { id: 1 },
  lock: {
    mode: "pessimistic_partial_write",
    tables: ["users"]
  }
});

// Lock with NOWAIT (fail immediately if locked)
await queryRunner.query(
  "SELECT * FROM users WHERE id = $1 FOR UPDATE NOWAIT",
  [userId]
);

// Lock with SKIP LOCKED (skip locked rows)
await queryRunner.query(
  "SELECT * FROM users WHERE status = 'pending' FOR UPDATE SKIP LOCKED LIMIT 1"
);
```

---

## 39. How do you handle database backups programmatically?

**Answer:**

```typescript
import { exec } from "child_process";
import { promisify } from "util";

const execAsync = promisify(exec);

// PostgreSQL backup
async function backupPostgreSQL() {
  const timestamp = new Date().toISOString().replace(/:/g, "-");
  const filename = `backup-${timestamp}.sql`;

  const command = `pg_dump -h localhost -U username -d database -f ${filename}`;

  try {
    await execAsync(command);
    console.log(`Backup created: ${filename}`);
  } catch (error) {
    console.error("Backup failed:", error);
  }
}

// MySQL backup
async function backupMySQL() {
  const timestamp = new Date().toISOString().replace(/:/g, "-");
  const filename = `backup-${timestamp}.sql`;

  const command = `mysqldump -h localhost -u username -p password database > ${filename}`;

  await execAsync(command);
  console.log(`Backup created: ${filename}`);
}

// Backup using TypeORM
async function exportData() {
  const users = await userRepository.find();
  const data = JSON.stringify(users, null, 2);

  await fs.writeFile(`users-backup-${Date.now()}.json`, data);
}

// Scheduled backups
import * as cron from "node-cron";

cron.schedule("0 2 * * *", async () => {
  // Run backup daily at 2 AM
  await backupPostgreSQL();
});

// Restore from backup
async function restore(filename: string) {
  const command = `psql -h localhost -U username -d database -f ${filename}`;
  await execAsync(command);
}
```

---

## 40. What are common TypeORM pitfalls and how to avoid them?

**Answer:**

**1. Using synchronize in production**
```typescript
// Wrong
synchronize: true // In production

// Right
synchronize: false,
migrations: ["dist/migration/**/*.js"],
migrationsRun: true
```

**2. N+1 query problem**
```typescript
// Wrong
const posts = await postRepository.find();
for (const post of posts) {
  console.log(post.author.name); // Extra query for each post
}

// Right
const posts = await postRepository.find({
  relations: ["author"]
});
```

**3. Not using transactions**
```typescript
// Wrong
await userRepository.save(user);
await accountRepository.save(account);

// Right
await AppDataSource.transaction(async manager => {
  await manager.save(user);
  await manager.save(account);
});
```

**4. Loading unnecessary data**
```typescript
// Wrong
const users = await userRepository.find(); // Loads all columns

// Right
const users = await userRepository.find({
  select: ["id", "name", "email"]
});
```

**5. Not handling connection errors**
**6. Circular dependencies in entities**
**7. Not using query builder for complex queries**
**8. Forgetting to close connections**
**9. Not indexing foreign keys**
**10. Using eager loading everywhere (performance impact)**

