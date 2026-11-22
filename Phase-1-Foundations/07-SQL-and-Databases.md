---
tags: [phase-1, sql, databases]
---
# 07 — SQL & Databases

> SQL is the lingua franca of enterprise data. As an FDE you'll live in customers' warehouses (Snowflake, BigQuery, Databricks, Postgres). **pgvector** puts vector search inside Postgres (Phase 5). Start with SQLite (zero setup) then Postgres.

## 1. Relational model
- **Table** (relation), **row** (tuple), **column** (attribute), **primary key** (unique id), **foreign key** (reference to another table), **constraints** (NOT NULL, UNIQUE, CHECK).
- **Normalization** (1NF–3NF): avoid duplicating facts; split into related tables. **Denormalization** for read speed/analytics.
- **ER diagrams**: entities & relationships (1–1, 1–many, many–many via join table).

## 2. Core SQL
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY, email TEXT UNIQUE NOT NULL, created_at TIMESTAMPTZ DEFAULT now());
CREATE TABLE orders (
  id SERIAL PRIMARY KEY, user_id INT REFERENCES users(id), total NUMERIC(10,2), status TEXT);

INSERT INTO users (email) VALUES ('a@x.com');
UPDATE orders SET status='paid' WHERE id=7;
DELETE FROM orders WHERE status='cancelled';

SELECT u.email, COUNT(*) AS n_orders, SUM(o.total) AS revenue
FROM users u
JOIN orders o ON o.user_id = u.id
WHERE o.status = 'paid' AND o.created_at >= DATE '2026-01-01'
GROUP BY u.email
HAVING SUM(o.total) > 1000
ORDER BY revenue DESC
LIMIT 10;
```
**Logical execution order:** `FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT`. (Explains why you can't use a SELECT alias in WHERE.)

### Joins
`INNER` (matches only) · `LEFT` (all left + matches) · `RIGHT` · `FULL OUTER` · `CROSS` · self-join · `ANTI-join` (`LEFT JOIN … WHERE r.id IS NULL` or `NOT EXISTS`) · `SEMI-join` (`EXISTS`).

### NULL
`NULL` = unknown. `NULL = NULL` is not true → use `IS NULL`. Aggregates ignore NULLs (except `COUNT(*)`). `COALESCE(x, 0)`.

## 3. Intermediate/advanced SQL
```sql
-- CTE
WITH paid AS (SELECT * FROM orders WHERE status='paid')
SELECT user_id, SUM(total) FROM paid GROUP BY user_id;

-- Window functions (interview favorite)
SELECT user_id, total,
  ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) AS rn,
  SUM(total) OVER (PARTITION BY user_id ORDER BY created_at) AS running_total,
  LAG(total) OVER (PARTITION BY user_id ORDER BY created_at) AS prev_total,
  RANK() OVER (ORDER BY total DESC) AS rnk
FROM orders;

-- Latest row per group
SELECT * FROM (SELECT *, ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY created_at DESC) rn FROM orders) t WHERE rn=1;

-- Recursive CTE (hierarchies)
WITH RECURSIVE tree AS (
  SELECT id, parent_id, 1 AS depth FROM categories WHERE parent_id IS NULL
  UNION ALL
  SELECT c.id, c.parent_id, t.depth+1 FROM categories c JOIN tree t ON c.parent_id=t.id)
SELECT * FROM tree;

-- Conditional aggregation / pivots
SELECT SUM(CASE WHEN status='paid' THEN total END) AS paid_total FROM orders;
```
Also: `UNION/UNION ALL/INTERSECT/EXCEPT`, subqueries (correlated), `CASE`, `DISTINCT ON` (Postgres), `LATERAL`, `GROUPING SETS/ROLLUP`, string/date functions, `JSONB` ops (`->`, `->>`, `@>`), full-text search (`tsvector`), arrays.

## 4. Transactions & ACID
- **Atomicity** (all or nothing) · **Consistency** (constraints hold) · **Isolation** (concurrent txs don't corrupt each other) · **Durability** (committed = persisted).
```sql
BEGIN; UPDATE acct SET bal=bal-100 WHERE id=1; UPDATE acct SET bal=bal+100 WHERE id=2; COMMIT;
```
- **Isolation levels:** Read Committed (PG default) → Repeatable Read → Serializable. Anomalies: dirty read, non-repeatable read, phantom read, lost update, write skew.
- **Locks, deadlocks, MVCC** (Postgres snapshots), optimistic vs pessimistic concurrency, `SELECT … FOR UPDATE`.

## 5. Indexes & performance
- **B-tree index** speeds equality/range lookups; cost: slower writes + disk.
- Composite index order matters (leftmost prefix). Covering indexes. Partial indexes. **GIN** (JSONB/full-text), **GiST**, **HNSW/IVFFlat** (pgvector).
- `EXPLAIN ANALYZE` → read the plan: Seq Scan vs Index Scan, join algorithms (nested loop/hash/merge), row estimates.
- Pitfalls: **N+1 queries** (ORM loop issuing a query per row), `SELECT *`, functions on indexed columns (`WHERE lower(email)=…` defeats index), missing `LIMIT`, unbounded `OFFSET`.
- Connection pooling (PgBouncer, SQLAlchemy pool).

## 6. Using DBs from Python
```python
import sqlite3
con = sqlite3.connect("app.db")
con.row_factory = sqlite3.Row
cur = con.execute("SELECT * FROM users WHERE email = ?", (email,))   # PARAMETERIZED — prevents SQL injection
```
- **Never** string-format user input into SQL.
- **ORMs/toolkits:** SQLAlchemy 2.x (Core + ORM), SQLModel, Django ORM, Peewee; **psycopg 3** / **asyncpg** for Postgres; **Alembic** for migrations.
- Migrations: version-controlled schema changes; forward-only in prod.

## 7. Database landscape
| Type | Examples | Use |
|---|---|---|
| Relational (OLTP) | Postgres, MySQL, SQL Server, SQLite | transactions, source of truth |
| Analytical (OLAP/warehouse) | Snowflake, BigQuery, Redshift, Databricks SQL, DuckDB, ClickHouse | big aggregations, columnar |
| Key-value / cache | Redis, DynamoDB, Memcached | fast lookups, sessions, rate limits |
| Document | MongoDB, Firestore | flexible schema |
| Graph | Neo4j, Neptune | relationships (GraphRAG) |
| Search | Elasticsearch/OpenSearch | BM25 full-text |
| **Vector** | pgvector, Qdrant, Pinecone, Weaviate, Milvus, Chroma | embeddings (Phase 5) |
| Object storage | S3, GCS, Azure Blob | files, parquet, datasets |
- **OLTP vs OLAP**, **ETL/ELT**, **data lake/lakehouse**, **dbt**, **Airflow/Dagster** orchestration, **CDC** (change data capture) — typical enterprise data plumbing you'll encounter as an FDE.
- **CAP theorem**: under a partition choose consistency or availability.

Next → [[08-Testing-Tooling-Docker]]
