# Chapter 22 — Advanced PostgreSQL Features

Features that make PostgreSQL much more than a traditional relational database: JSONB, Arrays, Full-Text Search, PostgreSQL Extensions. The important Senior Backend skill is knowing when to use each instead of blindly putting everything into JSONB.

---

## 1. JSONB

MongoDB:

```json
{ "_id": "...", "name": "Pulkit", "preferences": { "theme": "dark", "language": "en" } }
```

PostgreSQL:

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    preferences JSONB
);

INSERT INTO users (name, preferences)
VALUES ('Pulkit', '{"theme": "dark", "language": "en"}');
```

---

## 2. JSON vs JSONB

For most application use cases: prefer JSONB.

**JSON** — stores the JSON representation more directly.

**JSONB** — stores a binary/decomposed representation optimized for querying, and supports indexing much more effectively.

Application metadata, dynamic attributes, external API payloads, flexible configuration → JSONB.

---

## 3. Reading JSONB

```sql
SELECT preferences FROM users;

SELECT preferences -> 'theme' FROM users;   -- "dark" (JSON)

SELECT preferences ->> 'theme' FROM users;  -- dark (text)
```

```text
->   JSON
->>  TEXT
```

---

## 4. Nested JSON

```json
{ "notifications": { "email": true, "push": false } }
```

```sql
SELECT preferences -> 'notifications' ->> 'email'
FROM users;
```

Result: `true`

---

## 5. Filtering JSONB

```sql
SELECT *
FROM users
WHERE preferences @> '{"theme": "dark"}';
```

`@>` — does the left JSONB value contain the right JSON structure? A very important JSONB operator.

---

## 6. JSONB GIN Index

```sql
CREATE INDEX idx_users_preferences
ON users
USING GIN(preferences);
```

```text
JSONB → GIN index → Fast containment queries
```

---

## 7. When NOT to Use JSONB

Don't do `orders(id, data JSONB)` with everything (`userId`, `status`, `total`, `createdAt`, `items`) inside. Loses foreign keys, constraints, typed columns, efficient relational joins, clear schema.

Use `orders`, `users`, `order_items`, `products`, `payments` as proper tables, and JSONB only where flexibility is genuinely needed (see [Chapter 17 §15](17-advanced-postgresql-backend.md)).

---

## 8. Hybrid Schema

```text
users
----------------
id
email
name
status
created_at
metadata JSONB
```

Stable fields (`email`, `name`, `status`) remain proper columns; flexible data (`metadata`) is JSONB:

```json
{ "marketingSource": "instagram", "device": "iphone", "experiments": ["new_home", "checkout_v2"] }
```

A good use of JSONB.

---

## 9. Arrays

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    name TEXT,
    skills TEXT[]
);

INSERT INTO users (name, skills)
VALUES ('Pulkit', ARRAY['Node.js', 'PostgreSQL', 'NestJS']);
```

---

## 10. Query Arrays

```sql
SELECT * FROM users WHERE 'Node.js' = ANY(skills);

SELECT * FROM users WHERE skills @> ARRAY['Node.js'];
```

`@>` here: does this array contain `Node.js`?

---

## 11. Arrays vs Separate Table

**Array** — good when small, simple, rarely queried individually, no additional metadata.

**Separate table** — better when skills need relationships, indexes, metadata, are shared entities, or need joins:

```text
User → UserSkill → Skill
```

Better if skills are first-class entities.

---

## 12. Full-Text Search

```text
products(id, name, description)
```

Searching `"wireless bluetooth headphones"` — a plain `LIKE '%bluetooth%'` isn't ideal for serious text search. PostgreSQL has built-in full-text search.

---

## 13. tsvector

```sql
SELECT to_tsvector('english', 'Wireless Bluetooth Headphones');
```

Conceptually: `'bluetooth' 'headphon' 'wireless'` — note stemming (`headphones → headphon`).

---

## 14. tsquery

```sql
SELECT to_tsquery('english', 'wireless & bluetooth');
```

```sql
SELECT *
FROM products
WHERE to_tsvector('english', name || ' ' || description)
    @@ to_tsquery('english', 'wireless & bluetooth');
```

`@@` — does the document match the search query?

---

## 15. GIN Index for Full-Text Search

```sql
CREATE INDEX idx_products_search
ON products
USING GIN(to_tsvector('english', name || ' ' || description));
```

```text
Search query → GIN index → matching products
```

Much faster than recalculating `to_tsvector()` on every row for every search.

---

## 16. Generated Search Column

```sql
ALTER TABLE products
ADD COLUMN search_vector tsvector
GENERATED ALWAYS AS (
    to_tsvector('english', coalesce(name, '') || ' ' || coalesce(description, ''))
) STORED;

CREATE INDEX idx_products_search
ON products
USING GIN(search_vector);
```

```sql
SELECT *
FROM products
WHERE search_vector @@ websearch_to_tsquery('english', 'wireless headphones');
```

`websearch_to_tsquery()` is particularly convenient for user-entered search text.

---

## 17. Full-Text Search vs LIKE

| Requirement | Approach |
|---|---|
| Exact equality | `=` |
| Prefix search | B-tree in suitable cases |
| Simple substring | `LIKE` / `ILIKE` |
| Fuzzy matching | `pg_trgm` |
| Word-based search | Full-text search |
| Complex search/relevance | Elasticsearch/OpenSearch may be appropriate |

Don't automatically introduce Elasticsearch — PostgreSQL handles a lot of search workloads itself.

---

## 18. PostgreSQL Trigram

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE INDEX idx_users_name_trgm
ON users
USING GIN(name gin_trgm_ops);
```

```sql
SELECT * FROM users WHERE name ILIKE '%lkit%';
```

can use trigram indexing.

---

## 19. Similarity Search

```sql
SELECT name, similarity(name, 'Pulkit') AS score
FROM users
ORDER BY score DESC
LIMIT 10;
```

Useful for typo-tolerant search, autocomplete, names, product names, company names.

---

## 20. PostgreSQL Extensions

```sql
CREATE EXTENSION ...
```

Important ones: `pg_trgm`, PostGIS, `pgvector`, `uuid-ossp`, `citext`.

---

## 21. pgvector

Especially important for modern AI applications — stores vectors alongside data:

```sql
CREATE EXTENSION vector;

CREATE TABLE documents (
    id UUID PRIMARY KEY,
    content TEXT,
    embedding vector(1536)
);
```

Instead of storing raw text, store its embedding: `[0.013, -0.42, 0.18, ...]`.

---

## 22. Vector Similarity

```sql
SELECT id, content
FROM documents
ORDER BY embedding <=> $1
LIMIT 10;
```

```text
<->   Euclidean distance
<=>   cosine distance
```

The foundation of many RAG systems.

---

## 23. Vector Indexing

For large vector datasets: `HNSW`, `IVFFlat`.

```sql
CREATE INDEX idx_documents_embedding
ON documents
USING hnsw (embedding vector_cosine_ops);
```

Useful for RAG, semantic search, recommendation, document search, AI assistants inside PostgreSQL.

---

## 24. Why pgvector Is Interesting

```text
PostgreSQL
├── Users
├── Documents
├── Permissions
├── Metadata
└── Embeddings
```

vs maintaining PostgreSQL + a separate vector database — can simplify architecture for some workloads. At very large scale, specialized vector infrastructure may still be appropriate.

> Choose architecture based on workload, not hype.

---

## 25. CITEXT

Case-insensitive text behavior — useful for emails, instead of manually `LOWER(email) = LOWER($1)`. Be careful with email normalization rules; not every part of every email address can be normalized arbitrarily.

---

## 26. UUID Extensions

```sql
gen_random_uuid()
```

Older projects may use `uuid-ossp`. Don't add extensions just because old tutorials show them — prefer the simplest built-in functionality the PostgreSQL version provides.

---

## 27. JSONB + Vector = AI Application

```text
documents
--------------------------------
id
tenant_id
content
metadata JSONB
embedding vector(1536)
created_at
```

`metadata` might hold `{ "department": "finance", "documentType": "tender", "language": "en" }`, while `embedding` handles semantic similarity:

```text
User query → Embedding → Vector similarity → Filter tenant_id → Filter metadata → Top relevant documents
```

A powerful RAG architecture.

---

## 28. Multi-Tenant Vector Search

Tenants A, B, C sharing one database — never search without tenant isolation:

```sql
SELECT id, content
FROM documents
WHERE tenant_id = $2
ORDER BY embedding <=> $1
LIMIT 10;
```

Security filters must be part of the data-access design, not an afterthought (see [Chapter 10 §21-23](10-production-schema-design.md)).

---

## 29. JSONB + Relational + Vector

```text
documents
├── id              UUID
├── tenant_id       UUID
├── title           TEXT
├── content         TEXT
├── metadata        JSONB
├── search_vector   TSVECTOR
├── embedding       VECTOR
├── created_at      TIMESTAMPTZ
└── updated_at      TIMESTAMPTZ
```

```text
TEXT     → normal structured data
JSONB    → dynamic metadata
TSVECTOR → keyword/full-text search
VECTOR   → semantic search
```

A very powerful PostgreSQL design.

---

## 30. TypeORM + JSONB

```ts
@Column({ type: 'jsonb', nullable: true })
metadata: Record<string, unknown>;
```

```ts
const documents = await repository
  .createQueryBuilder('document')
  .where('document.metadata @> :metadata', {
    metadata: JSON.stringify({ department: 'finance' }),
  })
  .getMany();
```

The value is parameterized — never build `` `metadata @> '${userInput}'` `` manually.

---

## 31. Choosing the Right Data Type

| Data | Use |
|---|---|
| User email | `TEXT` |
| Price | `NUMERIC` / minor units |
| Status | `VARCHAR` + CHECK / enum |
| Tags, simple small list | `TEXT[]` |
| Dynamic metadata | `JSONB` |
| Relationships | Separate tables |
| Search text | `TSVECTOR` |
| Fuzzy text | `pg_trgm` |
| Embeddings | `VECTOR` / pgvector |
| Timestamp | `TIMESTAMPTZ` |

> Don't use JSONB simply because it's convenient. Model data according to how it will be queried, constrained, and maintained.

---

## 32. Interview question — "When would you use JSONB instead of a relational table?"

> I'd use JSONB for genuinely dynamic or semi-structured attributes that don't justify first-class relational modeling. For stable business entities, relationships, constraints, and frequently queried fields, I'd prefer normal columns and tables. If JSONB fields are queried frequently, I'd consider appropriate GIN or expression indexes.

---

## 33. Interview question — "PostgreSQL vs Elasticsearch for search?"

Not "Elasticsearch is always faster." Instead:

> PostgreSQL full-text search, trigram search, and indexes can handle many application search workloads without introducing another system. I'd consider Elasticsearch/OpenSearch when search requirements become substantially more advanced, such as large-scale distributed search, complex relevance tuning, specialized analyzers, or search workloads that justify a dedicated system.

---

## 34. Interview question — "Why would you use pgvector?"

> pgvector allows PostgreSQL to store and query embeddings alongside application data. This can simplify RAG and semantic-search architectures because relational filtering, permissions, metadata, and vector similarity can happen in the same database. For very large or specialized workloads, I'd evaluate dedicated vector infrastructure based on scale and performance requirements.

---

## 35. The Big Picture

```text
                     PostgreSQL
                         │
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
 Relational Data      JSONB           Search
       │                 │                 │
       ↓                 ↓                 ↓
 Users/Orders       Metadata        FTS/Trigram
       │
       └─────────────────┐
                         ↓
                     pgvector
                         ↓
                  Semantic Search
```

A powerful backend platform.

---

## Chapter 22 cheat sheet

```text
JSONB          → flexible structured data
GIN            → JSONB / full-text / some array searches
ARRAY          → small simple lists
Separate table → real relationships
TSVECTOR       → full-text search
pg_trgm        → fuzzy / substring search
pgvector       → embeddings / semantic search
HNSW           → approximate vector search
TypeORM        → abstraction, not replacement for SQL knowledge
```

---

## Next: Chapter 23 — Database Scaling & Architecture

Scaling from 1 server to 10 to 100, to millions/billions of rows: connection pooling → read replicas → replication lag → partitioning → sharding → PgBouncer → horizontal scaling → multi-tenancy → database bottlenecks → high-availability architecture.
