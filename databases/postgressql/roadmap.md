# PostgreSQL Learning Roadmap

> Find the detailed version at [roadmap.sh](https://roadmap.sh/postgresql-dba)

**Related:** [Backend](https://roadmap.sh/backend) | [MongoDB](https://roadmap.sh/mongodb) | [DevOps](https://roadmap.sh/devops)

---

## 1. Introduction

### What are Relational Databases?
- Structured data storage with table-based organization
- Relationships between data entities
- ACID compliance and SQL query language

### RDBMS Benefits and Limitations
**Benefits:** Data integrity, ACID transactions, structured queries, security
**Limitations:** Vertical scaling challenges, schema rigidity, complex joins

### PostgreSQL vs Other RDBMS
- PostgreSQL vs MySQL, Oracle, SQL Server
- Feature comparison and performance characteristics

### PostgreSQL vs NoSQL Databases
- Use case analysis and hybrid approaches
- PostgreSQL's JSON support for flexibility

---

## 2. Object Model & Basic RDBMS Concepts

### Relational Model
- **Domains**: Set of valid values
- **Attributes**: Column definitions
- **Tuples**: Individual rows
- **Relations**: Tables
- **Constraints**: Data integrity rules
- **NULL**: Absence of value

### Basic Concepts
- **Databases**: Top-level container
- **Schemas**: Logical namespaces
- **Tables**: Data structures
- **Columns**: Fields | **Rows**: Records
- **Queries**: SQL statements

### Data Types
Numeric, Character, Date/Time, Boolean, UUID, JSON/JSONB, Arrays, Geometric, Network address, Full-text search

---

## 3. High Level Database Concepts

### ACID Properties
**Atomicity** | **Consistency** | **Isolation** | **Durability**

### MVCC (Multi-Version Concurrency Control)
Non-blocking reads, snapshot isolation, transaction visibility

### Transactions
BEGIN, COMMIT, ROLLBACK, isolation levels, savepoints

### Write-ahead Log (WAL)
Transaction logging, crash recovery, PITR, replication

### Query Processing
Parsing → Planning → Optimization → Execution

---

## 4. Installation and Setup

### Installation Methods
- **Package Managers**: APT, YUM/DNF, Homebrew, Chocolatey
- **Using Docker**: Official images, Docker Compose
- **Cloud Deployment**: AWS RDS, Google Cloud SQL, Azure, Heroku, DigitalOcean

### Managing Postgres
- **systemd**: Modern Linux service management
- **pg_ctl**: PostgreSQL control utility
- **pg_ctlcluster**: Debian/Ubuntu management

### Connect using `psql`
Connection syntax, basic commands, .psqlrc configuration

---

## 5. Learn SQL

### DDL (Data Definition Language)

#### For Schemas
CREATE/ALTER/DROP SCHEMA, permissions

#### For Tables
CREATE/ALTER/DROP TABLE, constraints, keys

#### Data Types
Choosing types, custom types, casting, domains

### DML (Data Manipulation Language)

#### Querying Data
SELECT, DISTINCT, ORDER BY, LIMIT, OFFSET

#### Filtering Data
WHERE, comparison operators, LIKE/ILIKE, IN, BETWEEN, NULL checks, regex

#### Modifying Data
INSERT, UPDATE, DELETE, UPSERT, RETURNING

#### Joining Tables
INNER, LEFT, RIGHT, FULL OUTER, CROSS, self joins

### Advanced Querying

#### Subqueries
Scalar, row, table, correlated, EXISTS

#### Grouping
GROUP BY, HAVING, aggregates, GROUPING SETS, ROLLUP, CUBE

#### CTE (Common Table Expressions)
WITH clause, recursive CTEs

#### Lateral Join
LATERAL keyword, cross apply

#### Set Operations
UNION/UNION ALL, INTERSECT, EXCEPT

### Import / Export
**COPY TO/FROM**: CSV, binary formats, performance

---

## 6. Advanced SQL

### Procedures and Functions
CREATE FUNCTION/PROCEDURE, PL/pgSQL, parameters, return types

### Triggers
BEFORE/AFTER/INSTEAD OF triggers, event triggers

### Aggregate and Window Functions
Custom aggregates, ROW_NUMBER, RANK, PARTITION BY, LEAD/LAG

### Recursive CTE
Hierarchical queries, graph traversal, tree structures

---

## 7. Indexes and Use Cases

### Index Types

- **B-Tree**: Default, equality and range queries, sorted data
- **Hash**: Equality only, faster for exact matches
- **GiST**: Full-text search, geometric data
- **GIN**: Arrays, JSONB, full-text (faster searches than GiST)
- **SP-GiST**: Quad-trees, k-d trees, phone routing
- **BRIN**: Very large tables, time-series, minimal overhead

### Strategies
Composite, partial, unique indexes, maintenance

---

## 8. Configuring PostgreSQL

### postgres.conf Configuration

#### Resource Usage
shared_buffers, work_mem, maintenance_work_mem, effective_cache_size, max_connections

#### Write-ahead Log
wal_level, max_wal_size, wal_buffers, wal_compression

#### Checkpoints / Background Writer
checkpoint_timeout, checkpoint_completion_target, bgwriter settings

#### Query Planner
enable_seqscan, random_page_cost, cost parameters

#### Vacuums
autovacuum, workers, naptime, strategies

#### Replication
max_wal_senders, replication_slots, hot_standby

#### Reporting, Logging & Statistics
log_destination, logging_collector, log_statement, track_activities

### Adding Extensions
pg_stat_statements, pgcrypto, hstore, PostGIS, pg_trgm

### Per-User, Per-Database Settings
ALTER DATABASE/ROLE ... SET, precedence

### Storage Parameters
fillfactor, autovacuum_enabled, toast_tuple_target

---

## 9. Fine-grained Tuning

### Workload-Dependent Tuning

- **OLTP**: High concurrency, short transactions, connection pooling
- **OLAP**: Complex queries, parallel execution, materialized views
- **HTAP**: Mixed workloads, resource balancing

### Resource Usage / Capacity Planning
Hardware sizing, memory allocation, disk I/O, CPU, network, growth planning

---

## 10. Security

### Authentication Models
Password, SCRAM-SHA-256, peer, ident, certificate

### pg_hba.conf
Host-based auth, connection types, IP filtering

### Roles
CREATE ROLE, attributes, membership, privileges

### Object Privileges
Table, schema, database, function privileges

### Grant / Revoke
GRANT/REVOKE syntax, PUBLIC role

### Default Privileges
ALTER DEFAULT PRIVILEGES, schema and role defaults

### Row-Level Security
Policies, security barriers, performance

### SSL Settings
SSL modes, certificates, encryption

### SELinux
Security-Enhanced Linux integration, policies

---

## 11. Infrastructure Skills

### Replication

#### Streaming Replication
Physical replication, primary-standby, sync/async, cascading

#### Logical Replication
Publication/subscription, selective replication, multi-master

### Cluster Management

#### Patroni
HA solution, automatic failover, etcd/Consul/Zookeeper

#### Alternatives
repmgr, Stolon, pg_auto_failover, PAF

### Kubernetes Deployment
StatefulSets, persistent volumes, Helm charts, Operators (Zalando, Crunchy, CloudNativePG)

### Load Balancing / Discovery
**HAProxy**, **Consul**, **KeepAlived**, **Etcd**

### Monitoring
- **Prometheus**: postgres_exporter, Grafana
- **Zabbix**: Monitoring templates
- **Tools**: check_pgactivity, check_pgbackrest, temBoard, pgcenter

### Backup & Recovery

#### Built-in Tools
pg_dump, pg_dumpall, pg_restore, pg_basebackup

#### Third-party
pgbackrest, pg_probackup, barman, WAL-G

#### Validation
Regular restore testing, integrity checks

### Connection Pooling
**PgBouncer**: Session/transaction/statement pooling
**Alternatives**: pgpool-II, Odyssey, pgcat

### Anonymization
**PostgreSQL Anonymizer**: Data masking, GDPR compliance

### Upgrade Procedures
**pg_upgrade**: In-place upgrades, link/copy mode
**Logical Replication**: Zero-downtime upgrades

---

## 12. Application Skills

### Data and Processing

#### Bulk Loading
COPY command, pg_bulkload, parallel loading, ETL

#### Data Partitioning
Range, list, hash partitioning, pruning

#### Sharding Patterns
Horizontal partitioning, Citus, FDW

#### Normalization
1NF, 2NF, 3NF, BCNF, denormalization

#### Queues
LISTEN/NOTIFY, advisory locks, PgQ

#### Patterns / Antipatterns
Design patterns, common mistakes, schema/query best practices

### Migration Tools
Flyway, Liquibase, Alembic, Knex.js, sqitch

---

## 13. SQL Optimization

### Query Analysis
**EXPLAIN**, **Depesz**, **PEV2**, **Tensor**, **explain.dalibo.com**

### Optimization Techniques
Index usage, query rewriting, join optimization, avoiding SELECT *, statistics

### Monitoring Methodologies
**USE Method** (Utilization, Saturation, Errors)
**Golden Signals** (Latency, Traffic, Errors, Saturation)

---

## 14. Troubleshooting

### Postgres System Views
**pg_stat_activity**, **pg_stat_statements**, pg_stat_database, pg_locks, pg_stat_replication

### Tools
pgcenter, pgAdmin, DBeaver, psql

### Log Analysis
**pgBadger**, **pgCluu**, **awk/grep/sed**

### OS Tools
top, sysstat, iotop, vmstat, iostat

### Profiling
gdb, strace, perf-tools, eBPF, core dumps

---

## 15. Low Level Internals

### Architecture
Processes, memory, postmaster, backends, shared memory

### Core Concepts
- **Vacuum Processing**: Dead tuples, space reclamation, wraparound
- **Buffer Management**: Shared buffers, replacement strategies
- **Lock Management**: Lock types, deadlocks, advisory locks
- **Physical Storage**: Data directory, tablespaces, TOAST, FSM, VM
- **System Catalog**: pg_catalog, information_schema

---

## 16. Learn to Automate

### Scripting
Shell scripts, Python, Node.js, Java, Go

### Configuration Management
**Ansible**, **Salt**, **Puppet**, **Chef** - Infrastructure as Code

---

## 17. Get Involved

### Community
Mailing lists (pgsql-hackers, pgsql-general, pgsql-admin)

### Contribution
Reviewing patches, writing patches, CommitFest, Git workflow

---

## Resources

- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [PostgreSQL Wiki](https://wiki.postgresql.org/)
- [Planet PostgreSQL](https://planet.postgresql.org/)
- [PgExercises](https://pgexercises.com/)

---

## Learning Path

1. **Beginner**: Installation, basic SQL, CRUD, simple queries
2. **Intermediate**: Joins, indexes, transactions, administration, optimization
3. **Advanced**: Replication, backup/recovery, tuning, security
4. **Expert**: Internals, HA, automation, troubleshooting, development
5. **DBA**: Production management, monitoring, capacity planning, DR
