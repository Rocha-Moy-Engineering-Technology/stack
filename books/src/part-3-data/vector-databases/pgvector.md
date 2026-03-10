# pgvector

> PostgreSQL extension for vector similarity search and embeddings storage

| Field | Value |
|-------|-------|
| Name | pgvector |
| Group | Vector Databases |
| Type | SDK |
| Open Source | Yes |
| GitHub | [pgvector/pgvector](https://github.com/pgvector/pgvector) |
| Stars | 20,187 |
| Docs | [github.com/pgvector/pgvector](https://github.com/pgvector/pgvector) |

## Overview

pgvector is an open-source PostgreSQL extension that adds vector similarity search capabilities directly into PostgreSQL. It supports exact and approximate nearest neighbor search with single-precision, half-precision, binary, and sparse vector types. Rather than requiring a separate vector database, pgvector stores embedding vectors alongside relational data in the same database, leveraging PostgreSQL's ACID transactions, JOINs, indexing, query planning, Write-Ahead Log (WAL) replication, and point-in-time recovery. Written in C, it runs as a native in-process extension inside the PostgreSQL backend, avoiding network round-trips to external services. The extension requires PostgreSQL 13 or later and is available through Docker, Homebrew, APT, Yum, pkg, APK, conda-forge, PGXN, Postgres.app, and many managed cloud providers including Amazon RDS, Azure Database for PostgreSQL, Google Cloud SQL, Heroku Postgres, and Supabase.

## Core Concepts

### Vector Data Types

pgvector introduces four data types for different vector representations:

- **vector(n)**: Dense single-precision (32-bit float) vectors supporting up to 2,000 dimensions for indexed columns and up to 16,000 dimensions for unindexed storage. This is the primary type for embeddings from models such as OpenAI, Cohere, or sentence-transformers.
- **halfvec(n)**: Half-precision (16-bit float) vectors supporting up to 4,000 dimensions. Reduces storage by half while maintaining reasonable accuracy for most retrieval tasks.
- **bit(n)**: Binary vectors supporting up to 64,000 dimensions. Each dimension is a single bit, suitable for binary quantization schemes.
- **sparsevec(n)**: Sparse vectors supporting up to 1,000 non-zero elements. Efficient for high-dimensional vectors where most values are zero, such as TF-IDF or BM25 representations. Format: `'{0:0.1,10:0.2}'::sparsevec`.

All vectors in a column must have matching dimensions; mixed-dimension columns are not supported.

### Distance Functions

pgvector provides six distance operators, all returning lower values for greater similarity:

- **`<->` (L2 distance)**: Euclidean distance. Best general-purpose metric when vectors are not normalized.
- **`<#>` (negative inner product)**: Returns the negative dot product. Multiply by -1 to get actual inner product. Useful when vector magnitude carries meaningful signal.
- **`<=>` (cosine distance)**: Measures the angle between vectors, ignoring magnitude. Subtract from 1 to get cosine similarity. Preferred when vectors are normalized or magnitude should not influence results.
- **`<+>` (L1 distance)**: Manhattan distance. Sum of absolute differences across dimensions.
- **`<~>` (Hamming distance)**: Counts positions where corresponding bits differ. Operates on `bit` type only.
- **`<%>` (Jaccard distance)**: Measures dissimilarity between bit sets. Operates on `bit` type only.

### Indexing Strategies

Without an index, pgvector performs exact nearest neighbor search by scanning all rows, guaranteeing perfect recall but not scaling beyond small datasets. Two approximate nearest neighbor (ANN) index types trade recall for speed:

- **HNSW (Hierarchical Navigable Small World)**: A graph-based index implementing a multilayer navigable small world structure. Provides better query performance (higher recall at given latency) than IVFFlat but has slower build times and higher memory usage. No training step required -- indexes can be created on empty tables. The implementation follows the original HNSW paper algorithms for search, neighbor selection, and element insertion.
- **IVFFlat (Inverted File with Flat compression)**: A partition-based index that clusters vectors into lists using k-means, then searches the closest cluster subsets. Faster to build and uses less memory than HNSW but generally provides lower recall. Requires the table to already contain representative data before index creation for effective clustering.

## Installation

### From Source (Linux and Mac)

Requires PostgreSQL 13+ development headers:

```bash
cd /tmp
git clone --branch v0.8.2 https://github.com/pgvector/pgvector.git
cd pgvector
make
make install  # may need sudo
```

### From Source (Windows)

Requires Visual Studio with C++ support. Run from `x64 Native Tools Command Prompt` as administrator:

```batch
set "PGROOT=C:\Program Files\PostgreSQL\18"
cd %TEMP%
git clone --branch v0.8.2 https://github.com/pgvector/pgvector.git
cd pgvector
nmake /F Makefile.win
nmake /F Makefile.win install
```

### Package Managers

```bash
# Homebrew (macOS)
brew install pgvector

# APT (Debian/Ubuntu)
sudo apt install postgresql-17-pgvector

# Yum (RedHat/CentOS)
sudo yum install pgvector

# pkg (FreeBSD)
pkg install pgvector

# APK (Alpine Linux)
apk add pgvector

# PGXN
pgxn install vector

# conda-forge
conda install -c conda-forge pgvector
```

### Docker

```dockerfile
FROM pgvector/pgvector:pg17
```

Or add to an existing PostgreSQL image:

```dockerfile
FROM postgres:17
RUN apt-get update && apt-get install -y postgresql-17-pgvector
```

### Enabling the Extension

After installation, enable pgvector in each database where it is needed:

```sql
CREATE EXTENSION vector;
```

### Upgrading

Download and compile the newer version, then run:

```sql
ALTER EXTENSION vector UPDATE;
```

## Architecture

pgvector operates as a native PostgreSQL extension running within the PostgreSQL backend process:

- **In-process execution**: Vector operations run inside the PostgreSQL backend, avoiding network round-trips to external services. Distance calculations are implemented in C with CPU-dispatched SIMD optimizations on Linux x86-64.
- **WAL integration**: All vector data and index changes are written to the Write-Ahead Log, ensuring crash recovery and streaming replication work identically to standard PostgreSQL tables.
- **Planner integration**: The PostgreSQL query planner can combine vector index scans with B-tree index scans, filters, and joins in a single query plan. Cost estimation is tuned to help the planner select between sequential scan, vector index scan, and exact index scan depending on filter selectivity.
- **Shared buffer usage**: Vector data and indexes use PostgreSQL's shared buffer pool and benefit from the same caching, memory management, and vacuum processes as regular tables.
- **HNSW internals**: The HNSW implementation uses a multilayer graph structure with per-element neighbor arrays at each layer. Elements are stored as heap tuples with TID-based visited tracking for on-disk scans. The code implements the original paper's algorithms for search layer traversal (Algorithm 2), neighbor selection with pruning (Algorithm 4), and element insertion (Algorithm 1).
- **Storage**: Vector data uses `external` storage (stored out-of-line, not compressed), preventing TOAST compression overhead on vector columns.

## Key Features

- **Colocation of vectors and relational data**: Store embeddings in the same table as metadata, foreign keys, and other columns without external synchronization.
- **ACID transactions**: Vector inserts, updates, and deletes participate in PostgreSQL transactions with full consistency guarantees during concurrent writes.
- **JOIN support**: Combine vector similarity search with relational joins in a single query.
- **Six distance metrics**: L2, cosine, inner product, L1, Hamming, and Jaccard distances with corresponding operators and index support.
- **Two ANN index types**: HNSW for higher recall and IVFFlat for faster builds with lower memory consumption.
- **Iterative index scans (v0.8.0+)**: Automatically scan more of the index when filters reduce result count, available in `strict_order` mode for both HNSW and IVFFlat.
- **Half-precision vectors**: `halfvec` type halves storage while supporting all distance functions.
- **Sparse vector support**: `sparsevec` type for efficient storage of vectors with mostly zero values.
- **Binary quantization**: `binary_quantize()` function converts full-precision vectors to binary for compact storage and fast Hamming distance comparison.
- **Subvector indexing**: Create indexes on vector slices using expression indexes.
- **Parallel index builds**: Both HNSW and IVFFlat support parallel construction workers for faster index creation on multi-core systems.
- **Bulk loading**: COPY with binary format for high-throughput vector insertion.
- **Concatenation operator**: Combine vectors with the concatenation operator.
- **Aggregate functions**: `AVG()` and `SUM()` for vector columns.
- **Helper functions**: `vector_dims()`, `vector_norm()`, `cosine_similarity()`, `l2_normalize()`, `subvector()`, `hamming_distance()`, `jaccard_distance()`.

## Use Cases

- **Retrieval-Augmented Generation (RAG)**: Store document chunk embeddings and retrieve the most relevant chunks for a query embedding before passing them to a large language model.
- **Semantic search**: Find documents, products, or records by meaning rather than keyword matching.
- **Hybrid search**: Combine full-text search (`tsvector`/`tsquery`) with vector similarity search in a single PostgreSQL query using reciprocal rank fusion or other combination strategies.
- **Recommendation systems**: Compute similarity between user and item embeddings stored alongside transactional data.
- **Duplicate detection**: Identify near-duplicate records by finding vectors within a small distance threshold.
- **Image retrieval**: Store image embeddings and find visually similar images using cosine or L2 distance.
- **Classification**: Use nearest neighbor vectors with known labels to classify new items.

## API Reference

### Table Definition

```sql
CREATE TABLE documents (
    id BIGSERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(1536)
);

-- Add vector column to existing table
ALTER TABLE documents ADD COLUMN embedding vector(1536);
```

### Insert Vectors

```sql
-- Single insert
INSERT INTO documents (content, embedding)
VALUES ('Sample text', '[0.1, 0.2, 0.3, ...]');

-- Bulk upsert
INSERT INTO documents (id, content, embedding)
VALUES (1, 'text one', '[0.1, 0.2, 0.3]'), (2, 'text two', '[0.4, 0.5, 0.6]')
ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding;
```

### Nearest Neighbor Queries

```sql
-- L2 distance (Euclidean)
SELECT id, content, embedding <-> '[0.1, 0.2, 0.3]' AS distance
FROM documents
ORDER BY embedding <-> '[0.1, 0.2, 0.3]'
LIMIT 5;

-- Cosine similarity (1 - cosine distance)
SELECT id, content, 1 - (embedding <=> '[0.1, 0.2, 0.3]') AS similarity
FROM documents
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'
LIMIT 5;

-- Inner product (multiply by -1 since <#> returns negative)
SELECT id, content, (embedding <#> '[0.1, 0.2, 0.3]') * -1 AS inner_product
FROM documents
ORDER BY embedding <#> '[0.1, 0.2, 0.3]'
LIMIT 5;

-- Find nearest neighbors to an existing row
SELECT * FROM documents WHERE id != 1
ORDER BY embedding <-> (SELECT embedding FROM documents WHERE id = 1)
LIMIT 5;

-- Distance threshold query
SELECT id, content, embedding <=> '[0.1, 0.2, 0.3]' AS distance
FROM documents
WHERE embedding <=> '[0.1, 0.2, 0.3]' < 0.3
ORDER BY embedding <=> '[0.1, 0.2, 0.3]';
```

### Create Indexes

```sql
-- HNSW indexes
CREATE INDEX ON documents USING hnsw (embedding vector_l2_ops);
CREATE INDEX ON documents USING hnsw (embedding vector_ip_ops);
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON documents USING hnsw (embedding vector_l1_ops);
CREATE INDEX ON documents USING hnsw (embedding bit_hamming_ops);
CREATE INDEX ON documents USING hnsw (embedding bit_jaccard_ops);

-- HNSW with custom parameters
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

-- IVFFlat indexes
CREATE INDEX ON documents USING ivfflat (embedding vector_l2_ops) WITH (lists = 100);
CREATE INDEX ON documents USING ivfflat (embedding vector_ip_ops) WITH (lists = 100);
CREATE INDEX ON documents USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
CREATE INDEX ON documents USING ivfflat (embedding bit_hamming_ops) WITH (lists = 100);

-- Subvector index
CREATE INDEX ON documents USING hnsw ((embedding[1:100]) vector_l2_ops);

-- Partial index for filtered queries
CREATE INDEX ON documents USING hnsw (embedding vector_l2_ops) WHERE (category_id = 123);
```

### Index Operator Classes

HNSW supported types: `vector` (up to 2,000 dimensions), `halfvec` (up to 4,000 dimensions), `bit` (up to 64,000 dimensions), `sparsevec` (up to 1,000 non-zero elements).

IVFFlat supported types: `vector` (up to 2,000 dimensions), `halfvec` (up to 4,000 dimensions), `bit` (up to 64,000 dimensions).

Operator classes by distance function:

- L2: `vector_l2_ops`, `halfvec_l2_ops`
- Inner product: `vector_ip_ops`, `halfvec_ip_ops`
- Cosine: `vector_cosine_ops`, `halfvec_cosine_ops`
- L1: `vector_l1_ops`, `halfvec_l1_ops`
- Hamming: `bit_hamming_ops`
- Jaccard: `bit_jaccard_ops`

### Half-Precision Vectors

```sql
CREATE TABLE items (id bigserial PRIMARY KEY, embedding halfvec(3));
INSERT INTO items (embedding) VALUES ('[0.1, 0.2, 0.3]');
CREATE INDEX ON items USING hnsw (embedding halfvec_cosine_ops);
```

### Binary Vectors

```sql
CREATE TABLE items (id bigserial PRIMARY KEY, embedding bit(8));
INSERT INTO items (embedding) VALUES ('10101010'), ('11001100');

-- Binary quantization from full-precision vectors
SELECT binary_quantize(embedding) FROM documents;
```

### Sparse Vectors

```sql
CREATE TABLE items (id bigserial PRIMARY KEY, embedding sparsevec(1000));
INSERT INTO items (embedding) VALUES ('{0:0.1,10:0.2}'::sparsevec);
```

### Aggregate and Helper Functions

```sql
-- Aggregates
SELECT AVG(embedding) FROM documents;
SELECT SUM(embedding) FROM documents;
SELECT category_id, AVG(embedding) FROM documents GROUP BY category_id;

-- Helper functions
SELECT vector_dims(embedding) FROM documents LIMIT 1;
SELECT vector_norm(embedding) FROM documents LIMIT 1;
SELECT cosine_similarity(a.embedding, b.embedding) FROM documents a, documents b WHERE a.id = 1 AND b.id = 2;
SELECT l2_normalize(embedding) FROM documents LIMIT 1;
SELECT subvector(embedding, 1, 100) FROM documents LIMIT 1;
```

## Configuration

### HNSW Parameters

Index build parameters (set at creation time):

- **`m`** (default 16): Maximum number of connections per node in each layer. Higher values improve recall but increase memory and build time.
- **`ef_construction`** (default 64): Size of the dynamic candidate list during index construction. Higher values improve recall at the cost of slower builds.

Query-time parameters (set per session or transaction):

- **`hnsw.ef_search`** (default 40): Size of the dynamic candidate list during search. Higher values improve recall at the cost of higher latency.
- **`hnsw.iterative_scan`** (default `off`): Enable iterative scans with `strict_order` to automatically scan more of the index when filters reduce result count.

```sql
SET hnsw.ef_search = 100;
SET hnsw.iterative_scan = strict_order;

-- Transaction-scoped setting
BEGIN;
SET LOCAL hnsw.ef_search = 200;
SELECT ...;
COMMIT;
```

### IVFFlat Parameters

Index build parameter:

- **`lists`**: Number of inverted lists (clusters). Starting point: `rows / 1000` for up to 1M rows, `sqrt(rows)` for over 1M rows.

Query-time parameters:

- **`ivfflat.probes`** (default 1): Number of lists to search. Starting point: `sqrt(lists)`. Setting to the total number of lists produces exact search (planner will not use the index).
- **`ivfflat.iterative_scan`** (default `off`): Enable with `strict_order` for automatic additional scanning.

```sql
SET ivfflat.probes = 10;
SET ivfflat.iterative_scan = strict_order;
```

### Index Build Performance

```sql
-- Increase work memory for faster HNSW builds (graph must fit in memory)
SET maintenance_work_mem = '8GB';

-- Parallelize index construction (both HNSW and IVFFlat)
SET max_parallel_maintenance_workers = 7;  -- plus leader

-- May also need to increase max_parallel_workers (default: 8)
SET max_parallel_workers = 15;
```

A notice appears when the HNSW graph no longer fits in `maintenance_work_mem`:

```
NOTICE: hnsw graph no longer fits into maintenance_work_mem after 100000 tuples
DETAIL: Building will take significantly more time.
HINT: Increase maintenance_work_mem to speed up builds.
```

### Index Build Progress Monitoring

```sql
-- Check progress during index creation
SELECT phase, round(100.0 * blocks_done / nullif(blocks_total, 0), 1) AS "%"
FROM pg_stat_progress_create_index;
```

HNSW phases: `initializing`, `loading tuples`.

IVFFlat phases: `initializing`, `performing k-means`, `assigning tuples`, `loading tuples` (percentage only populated during `loading tuples`).

### General Recommendations

- Create indexes after loading initial data for better performance.
- For HNSW, increase `hnsw.ef_search` to improve recall at the cost of higher latency.
- For IVFFlat, increase `ivfflat.probes` to improve recall; use `sqrt(lists)` as a starting point.
- Set `maintenance_work_mem` to at least 1-8 GB when building indexes on large tables.
- Use `max_parallel_maintenance_workers` to speed up index creation on multi-core systems.
- Use COPY with binary format for bulk data loading.
- For filtered queries, create B-tree indexes on filter columns; enable iterative scans for approximate indexes with filters.
- For few distinct filter values, use partial indexes; for many values, use table partitioning.

## Integration Patterns

### Python with psycopg3

```python
import psycopg

conn = psycopg.connect("dbname=mydb")
conn.execute("CREATE EXTENSION IF NOT EXISTS vector")
conn.execute("""
    CREATE TABLE IF NOT EXISTS documents (
        id BIGSERIAL PRIMARY KEY,
        content TEXT,
        embedding vector(1536)
    )
""")

# Insert
embedding = [0.1, 0.2, 0.3]  # truncated
conn.execute(
    "INSERT INTO documents (content, embedding) VALUES (%s, %s)",
    ("sample text", str(embedding))
)

# Query nearest neighbors
query_embedding = [0.1, 0.2, 0.3]
results = conn.execute(
    "SELECT id, content FROM documents ORDER BY embedding <=> %s LIMIT 5",
    (str(query_embedding),)
).fetchall()
```

### Python with pgvector-python

The `pgvector` Python package (`pip install pgvector`) provides native type support for multiple drivers:

```python
from pgvector.psycopg import register_vector
import psycopg
import numpy as np

conn = psycopg.connect("dbname=mydb")
register_vector(conn)

# Insert with numpy array (no manual string conversion)
embedding = np.array([0.1, 0.2, 0.3])
conn.execute(
    "INSERT INTO documents (content, embedding) VALUES (%s, %s)",
    ("sample text", embedding)
)
```

Supported drivers: psycopg3, psycopg2, asyncpg, pg8000.

### SQLAlchemy

```python
from pgvector.sqlalchemy import Vector
from sqlalchemy import Column, Integer, Text, create_engine
from sqlalchemy.orm import declarative_base, Session

Base = declarative_base()

class Document(Base):
    __tablename__ = "documents"
    id = Column(Integer, primary_key=True)
    content = Column(Text)
    embedding = Column(Vector(1536))

engine = create_engine("postgresql://user:password@localhost/mydb")
Base.metadata.create_all(engine)

with Session(engine) as session:
    session.add(Document(content="text", embedding=[0.1, 0.2, 0.3]))
    session.commit()

    # Nearest neighbor query
    from pgvector.sqlalchemy import Vector
    results = session.query(Document).order_by(
        Document.embedding.cosine_distance([0.1, 0.2, 0.3])
    ).limit(5).all()
```

### Django

```python
from pgvector.django import VectorExtension, VectorField, HnswIndex

# Migration
class Migration(migrations.Migration):
    operations = [VectorExtension()]

# Model
class Document(models.Model):
    content = models.TextField()
    embedding = VectorField(dimensions=1536)

    class Meta:
        indexes = [HnswIndex(fields=['embedding'], opclasses=['vector_cosine_ops'])]
```

### LangChain

```python
from langchain_community.vectorstores import PGVector

connection_string = "postgresql://user:password@localhost:5432/mydb"
vectorstore = PGVector.from_documents(
    documents=docs,
    embedding=embeddings_model,
    connection_string=connection_string,
    collection_name="my_collection",
)
results = vectorstore.similarity_search("query text", k=5)
```

### Other Language Libraries

pgvector works with any language that has a PostgreSQL client. Official client libraries exist for Ruby, JavaScript/TypeScript (node-postgres, Knex.js, Objection.js, Sequelize, Prisma), PHP (Laravel), Go, Java (JDBC, Spring), Rust, .NET (Npgsql, Entity Framework Core), Elixir (Ecto), and Lua.

## Examples

### Basic RAG Pipeline

```sql
CREATE TABLE chunks (
    id BIGSERIAL PRIMARY KEY,
    document_id INTEGER REFERENCES documents(id),
    chunk_text TEXT NOT NULL,
    embedding vector(1536),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX chunks_embedding_idx ON chunks
USING hnsw (embedding vector_cosine_ops);

-- Retrieve top 5 chunks for a query embedding
SELECT chunk_text, 1 - (embedding <=> :query_embedding) AS similarity
FROM chunks
WHERE document_id IN (SELECT id FROM documents WHERE project_id = :project_id)
ORDER BY embedding <=> :query_embedding
LIMIT 5;
```

### Hybrid Search with Full-Text and Vector

```sql
CREATE TABLE articles (
    id BIGSERIAL PRIMARY KEY,
    title TEXT,
    body TEXT,
    tsv TSVECTOR GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || body)) STORED,
    embedding vector(1536)
);

CREATE INDEX articles_tsv_idx ON articles USING gin(tsv);
CREATE INDEX articles_embedding_idx ON articles USING hnsw (embedding vector_cosine_ops);

-- Reciprocal Rank Fusion combining keyword and semantic search
WITH keyword_results AS (
    SELECT id, ROW_NUMBER() OVER (ORDER BY ts_rank(tsv, plainto_tsquery('english', :query)) DESC) AS rank
    FROM articles
    WHERE tsv @@ plainto_tsquery('english', :query)
    LIMIT 20
),
vector_results AS (
    SELECT id, ROW_NUMBER() OVER (ORDER BY embedding <=> :query_embedding ASC) AS rank
    FROM articles
    ORDER BY embedding <=> :query_embedding
    LIMIT 20
),
combined AS (
    SELECT COALESCE(k.id, v.id) AS id,
           COALESCE(1.0 / (60 + k.rank), 0) + COALESCE(1.0 / (60 + v.rank), 0) AS rrf_score
    FROM keyword_results k
    FULL OUTER JOIN vector_results v ON k.id = v.id
)
SELECT a.id, a.title, c.rrf_score
FROM combined c
JOIN articles a ON a.id = c.id
ORDER BY c.rrf_score DESC
LIMIT 10;
```

### Binary Quantization with Re-ranking

```sql
-- Create binary quantized column for fast initial retrieval
ALTER TABLE documents ADD COLUMN embedding_binary bit(1536)
    GENERATED ALWAYS AS (binary_quantize(embedding)::bit(1536)) STORED;

CREATE INDEX ON documents USING hnsw (embedding_binary bit_hamming_ops);

-- Two-stage retrieval: fast binary search then re-rank with full precision
WITH candidates AS (
    SELECT id, content, embedding
    FROM documents
    ORDER BY embedding_binary <~> binary_quantize(:query_embedding)::bit(1536)
    LIMIT 100
)
SELECT id, content, 1 - (embedding <=> :query_embedding) AS similarity
FROM candidates
ORDER BY embedding <=> :query_embedding
LIMIT 10;
```

### Filtered Vector Search with Iterative Scans

```sql
-- Enable iterative scans for better filtered search
SET hnsw.iterative_scan = strict_order;

SELECT id, content
FROM documents
WHERE category = 'technical' AND created_at > '2025-01-01'
ORDER BY embedding <=> :query_embedding
LIMIT 10;
```

### Partitioned Table for Scaling

```sql
CREATE TABLE items (
    id BIGSERIAL,
    embedding vector(1536),
    category_id INT
) PARTITION BY LIST(category_id);

CREATE TABLE items_cat_1 PARTITION OF items FOR VALUES IN (1);
CREATE TABLE items_cat_2 PARTITION OF items FOR VALUES IN (2);

-- Create per-partition indexes
CREATE INDEX ON items_cat_1 USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON items_cat_2 USING hnsw (embedding vector_cosine_ops);
```

## Limitations

- **vector(n) dimension limit**: Dense vector indexes are limited to 2,000 dimensions. Unindexed vector columns support up to 16,000 dimensions. Models producing higher-dimensional embeddings require dimensionality reduction or `halfvec` (up to 4,000 indexed dimensions).
- **Approximate recall**: HNSW and IVFFlat indexes provide approximate results. Recall depends on index parameters and may not reach 100% without exact (sequential) scan.
- **HNSW build time**: HNSW index construction can be slow on large datasets (millions of vectors) and requires the graph to fit in `maintenance_work_mem` for optimal build speed.
- **HNSW memory consumption**: HNSW indexes reside in memory and can be substantial. Sizing depends on number of vectors, dimensionality, and the `m` parameter. Indexes can exceed memory but performance degrades with disk access.
- **IVFFlat requires pre-populated data**: Building an IVFFlat index on an empty or very small table produces poor clusters. The table should contain a representative sample of data before index creation.
- **No built-in sharding**: pgvector relies on PostgreSQL's native partitioning and external sharding solutions (such as Citus) for horizontal scaling. It does not provide built-in distributed vector search.
- **Single-node scaling**: Performance is bounded by single-node PostgreSQL limits. For datasets exceeding hundreds of millions of vectors, purpose-built distributed vector databases may offer better throughput.
- **Fixed dimensions per column**: All vectors in a column must have the same number of dimensions. Mixed-dimension storage requires separate columns or tables.
- **sparsevec index support**: Sparse vectors support HNSW indexing only (not IVFFlat) and are limited to L2, inner product, and cosine distances.

## Changelog

- **v0.8.2** (2026-02-25): Fixed buffer overflow with parallel HNSW index build. Improved Windows install target. Fixed EXPLAIN output for Postgres 18.
- **v0.8.1** (2025-09-04): Added support for Postgres 18 rc1. Improved `binary_quantize` performance.
- **v0.8.0** (2024-10-30): Added iterative index scans for both HNSW and IVFFlat. Added array-to-sparsevec casts. Improved cost estimation for filtered queries. Improved HNSW scan, insert, and on-disk build performance. Dropped Postgres 12 support.
- **v0.7.0** (2024-04-29): Added `halfvec` and `sparsevec` types. Added `bit` type indexing. Added L1 distance indexing for HNSW. Added `binary_quantize`, `hamming_distance`, `jaccard_distance`, `l2_normalize`, `subvector` functions. Added vector concatenation operator. Added CPU dispatching for distance functions on Linux x86-64.
- **v0.6.0** (2024-01-29): Added parallel HNSW index builds. Changed vector storage from `extended` to `external`. Improved HNSW performance and reduced WAL generation. Moved Docker image to `pgvector` org. Dropped Postgres 11 support.
- **v0.5.0** (2023-08-28): Added HNSW index type. Added parallel IVFFlat builds. Added `l1_distance` function, element-wise multiplication, `sum` aggregate. Improved distance function performance.
- **v0.4.0** (2023-01-11): Increased max vector dimensions from 1,024 to 16,000 (indexed: 2,000). Changed storage from `plain` to `extended`. Added `avg` aggregate. Added experimental Windows support. Dropped Postgres 10 support.
- **v0.3.0** (2022-10-15): Added Postgres 15 support. Dropped Postgres 9.6 support.
- **v0.1.0** (2021-04-20): First release with IVFFlat indexing, L2/inner product/cosine distance operators, and basic vector type.

## Citations

- [1] pgvector GitHub repository and README - https://github.com/pgvector/pgvector
- [2] pgvector Changelog - https://github.com/pgvector/pgvector/blob/master/CHANGELOG.md
- [3] pgvector-python client library - https://github.com/pgvector/pgvector-python
- [4] pgvector HNSW implementation (hnswutils.c) - https://github.com/pgvector/pgvector/blob/master/src/hnswutils.c
- [5] pgvector GitHub API metadata - https://api.github.com/repos/pgvector/pgvector
