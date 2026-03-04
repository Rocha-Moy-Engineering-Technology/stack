# pgvector

> PostgreSQL extension for vector similarity search, enabling storage and retrieval of embeddings alongside relational data with full ACID compliance.

| Field | Value |
|-------|-------|
| Group | RAG & Knowledge Retrieval |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/pgvector/pgvector](https://github.com/pgvector/pgvector) |
| Stars | 19914 |
| Documentation | [Official Docs](https://github.com/pgvector/pgvector) |

## Overview

pgvector is a PostgreSQL extension that adds vector similarity search capabilities directly into PostgreSQL. Rather than requiring a separate vector database, pgvector allows developers to store embedding vectors alongside relational data in the same database, leveraging PostgreSQL's mature ecosystem of ACID transactions, JOINs, indexing, and query planning. This eliminates the operational overhead of synchronizing data between a relational database and a dedicated vector store, making it a practical choice for applications that need both structured queries and semantic search.

## Core Concepts

### Vector Data Types

pgvector introduces several data types for storing different representations of vectors:

- **vector(n)**: Dense floating-point vectors supporting up to 2,000 dimensions. This is the primary type used for storing embeddings from models such as OpenAI, Cohere, or sentence-transformers.
- **halfvec(n)**: Half-precision floating-point vectors supporting up to 4,000 dimensions. Uses 16-bit floats to reduce storage while maintaining reasonable accuracy for many retrieval tasks.
- **bit(n)**: Binary vectors supporting up to 64,000 dimensions. Suitable for binary quantization schemes where each dimension is represented as a single bit.
- **sparsevec(n)**: Sparse vectors supporting up to 1,000 non-zero elements. Efficient for high-dimensional vectors where most values are zero, such as TF-IDF or BM25 representations.

### Distance Functions

pgvector provides six distance operators for computing similarity between vectors:

- **`<->` (L2 distance)**: Euclidean distance. Lower values indicate greater similarity. Best general-purpose metric when vectors are not normalized.
- **`<#>` (negative inner product)**: Returns the negative dot product. Lower values indicate greater similarity. Useful when vectors encode magnitude as meaningful signal.
- **`<=>` (cosine distance)**: Measures the angle between vectors, ignoring magnitude. Lower values indicate greater similarity. Preferred when vectors are normalized or when magnitude should not influence results.
- **`<+>` (L1 distance)**: Manhattan distance. Sum of absolute differences across dimensions. Lower values indicate greater similarity.
- **`<~>` (Hamming distance)**: Counts the number of positions where corresponding bits differ. Operates on `bit` type vectors.
- **`<%>` (Jaccard distance)**: Measures dissimilarity between bit sets. Operates on `bit` type vectors.

### Indexing Strategies

pgvector supports two approximate nearest neighbor (ANN) index types:

- **HNSW (Hierarchical Navigable Small World)**: A graph-based index that provides better recall at the cost of higher memory usage and slower build times. Default parameters are `m=16` (max connections per node) and `ef_construction=64` (size of the dynamic candidate list during construction). At query time, `hnsw.ef_search=40` controls the search breadth. Increasing `ef_search` improves recall but increases latency.
- **IVFFlat (Inverted File with Flat compression)**: A partition-based index that clusters vectors into lists. Faster to build and uses less memory than HNSW but generally provides lower recall. The `lists` parameter controls the number of clusters, and `ivfflat.probes` controls how many clusters are searched at query time.

Without an index, pgvector performs exact nearest neighbor search by scanning all rows, which guarantees perfect recall but does not scale beyond small datasets.

## Installation and Setup

### From Source

```bash
git clone --branch v0.8.0 https://github.com/pgvector/pgvector.git
cd pgvector
make
make install
```

This requires PostgreSQL development headers (`postgresql-server-dev-*` on Debian/Ubuntu or the equivalent for your platform).

### Package Managers

```bash
# Homebrew (macOS)
brew install pgvector

# APT (Debian/Ubuntu)
sudo apt install postgresql-17-pgvector

# PGXN
pgxn install vector
```

### Docker

```dockerfile
FROM pgvector/pgvector:pg17
```

Or add pgvector to an existing PostgreSQL image:

```dockerfile
FROM postgres:17
RUN apt-get update && apt-get install -y postgresql-17-pgvector
```

### Enabling the Extension

After installation, enable pgvector in your database:

```sql
CREATE EXTENSION vector;
```

## Architecture

pgvector operates as a native PostgreSQL extension, meaning it runs within the PostgreSQL process and integrates directly with the query planner, executor, and storage engine. Key architectural characteristics include:

- **In-process execution**: Vector operations run inside the PostgreSQL backend process, avoiding network round-trips to external services.
- **WAL integration**: All vector data and index changes are written to the Write-Ahead Log (WAL), ensuring crash recovery and replication work identically to standard PostgreSQL tables.
- **Planner integration**: The PostgreSQL query planner can combine vector index scans with other index scans, filters, and joins in a single query plan.
- **Shared buffer usage**: Vector data and indexes use PostgreSQL's shared buffer pool, benefiting from the same caching and memory management as regular tables.

## Key Features and Functionality

- **Colocation of vectors and relational data**: Store embeddings in the same table as metadata, foreign keys, and other columns. No external synchronization required.
- **ACID transactions**: Vector inserts, updates, and deletes participate in PostgreSQL transactions, ensuring consistency even during concurrent writes.
- **JOIN support**: Combine vector similarity search with relational joins, enabling queries such as "find the most similar documents written by a specific author."
- **Multiple distance metrics**: Six built-in distance operators covering Euclidean, cosine, inner product, Manhattan, Hamming, and Jaccard distances.
- **Two ANN index types**: HNSW for higher recall and IVFFlat for faster builds with lower memory consumption.
- **Iterative scans (v0.8.0+)**: `strict_order` and `relaxed_order` scan modes allow the query planner to interleave index scans with filter evaluation, improving performance when combining vector search with `WHERE` clauses.
- **Half-precision and sparse vector support**: Reduce storage with `halfvec` or efficiently store sparse representations with `sparsevec`.

## Use Cases

- **Retrieval-Augmented Generation (RAG)**: Store document chunk embeddings and retrieve the most relevant chunks for a given query embedding before passing them to a large language model (LLM).
- **Semantic search**: Find documents, products, or records by meaning rather than keyword matching.
- **Recommendation systems**: Compute similarity between user and item embeddings stored alongside transactional data.
- **Duplicate detection**: Identify near-duplicate records by finding vectors within a small distance threshold.
- **Hybrid search**: Combine full-text search (`tsvector`) with vector similarity search in a single PostgreSQL query.
- **Image retrieval**: Store image embeddings and find visually similar images using cosine or L2 distance.

## API Reference Summary

### Table Definition

```sql
CREATE TABLE documents (
    id BIGSERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(1536)
);
```

### Insert Vectors

```sql
INSERT INTO documents (content, embedding)
VALUES ('Sample text', '[0.1, 0.2, 0.3, ...]');
```

### Nearest Neighbor Queries

```sql
-- L2 distance (Euclidean)
SELECT id, content, embedding <-> '[0.1, 0.2, 0.3]' AS distance
FROM documents
ORDER BY embedding <-> '[0.1, 0.2, 0.3]'
LIMIT 5;

-- Cosine distance
SELECT id, content, 1 - (embedding <=> '[0.1, 0.2, 0.3]') AS similarity
FROM documents
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'
LIMIT 5;

-- Inner product (returns negative, so lower is more similar)
SELECT id, content, embedding <#> '[0.1, 0.2, 0.3]' AS neg_inner_product
FROM documents
ORDER BY embedding <#> '[0.1, 0.2, 0.3]'
LIMIT 5;
```

### Create Indexes

```sql
-- HNSW index with cosine distance
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

-- IVFFlat index with L2 distance
CREATE INDEX ON documents
USING ivfflat (embedding vector_l2_ops)
WITH (lists = 100);
```

### Index Operator Classes

| Distance      | Operator | HNSW Operator Class   | IVFFlat Operator Class |
|---------------|----------|-----------------------|------------------------|
| L2            | `<->`    | `vector_l2_ops`       | `vector_l2_ops`        |
| Inner product | `<#>`    | `vector_ip_ops`       | `vector_ip_ops`        |
| Cosine        | `<=>`    | `vector_cosine_ops`   | `vector_cosine_ops`    |
| L1            | `<+>`    | `vector_l1_ops`       | `vector_l1_ops`        |

### Filtering with Vector Search

```sql
SELECT id, content
FROM documents
WHERE category = 'technical'
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'
LIMIT 10;
```

### Iterative Scans (v0.8.0+)

```sql
-- Strict ordering: guarantees results are in exact distance order
SET hnsw.iterative_scan = strict_order;

SELECT id, content
FROM documents
WHERE category = 'technical'
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'
LIMIT 10;

-- Relaxed ordering: faster but may return slightly out-of-order results
SET hnsw.iterative_scan = relaxed_order;
```

### Aggregate Functions

```sql
-- Average of vectors
SELECT AVG(embedding) FROM documents;

-- Sum of vectors
SELECT SUM(embedding) FROM documents;
```

## Configuration and Customization

### Index Build Performance

```sql
-- Increase work memory for faster index builds
SET maintenance_work_mem = '2GB';

-- Parallelize index construction
SET max_parallel_maintenance_workers = 7;
```

### HNSW Parameters

| Parameter             | Default | Description                                          |
|-----------------------|---------|------------------------------------------------------|
| `m`                   | 16      | Maximum number of connections per node in the graph   |
| `ef_construction`     | 64      | Size of the dynamic candidate list during index build |
| `hnsw.ef_search`      | 40      | Size of the dynamic candidate list during query       |
| `hnsw.iterative_scan` | off     | Enable iterative scan (`strict_order` or `relaxed_order`) |

### IVFFlat Parameters

| Parameter         | Default | Description                                                            |
|-------------------|---------|------------------------------------------------------------------------|
| `lists`           | -       | Number of inverted lists (clusters). Typical starting point: `sqrt(N)` |
| `ivfflat.probes`  | 1       | Number of lists to search at query time                                |

### General Recommendations

- For HNSW, increase `hnsw.ef_search` to improve recall at the cost of higher latency.
- For IVFFlat, increase `ivfflat.probes` to improve recall. A common starting point for `lists` is the square root of the total number of rows.
- Set `maintenance_work_mem` to at least 1 GB when building indexes on large tables.
- Use `max_parallel_maintenance_workers` to speed up index creation on multi-core systems.

## Integration Patterns

### Python with psycopg

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
embedding = [0.1, 0.2, 0.3]  # truncated for brevity
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

```python
from pgvector.psycopg import register_vector
import psycopg
import numpy as np

conn = psycopg.connect("dbname=mydb")
register_vector(conn)

embedding = np.array([0.1, 0.2, 0.3])
conn.execute(
    "INSERT INTO documents (content, embedding) VALUES (%s, %s)",
    ("sample text", embedding)
)
```

### LangChain Integration

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

### SQLAlchemy with pgvector

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
```

## Examples

### Basic RAG Pipeline

```sql
-- Create table for document chunks
CREATE TABLE chunks (
    id BIGSERIAL PRIMARY KEY,
    document_id INTEGER REFERENCES documents(id),
    chunk_text TEXT NOT NULL,
    embedding vector(1536),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create HNSW index for cosine similarity
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
-- Table with both tsvector and vector columns
CREATE TABLE articles (
    id BIGSERIAL PRIMARY KEY,
    title TEXT,
    body TEXT,
    tsv TSVECTOR GENERATED ALWAYS AS (to_tsvector('english', title || ' ' || body)) STORED,
    embedding vector(1536)
);

CREATE INDEX articles_tsv_idx ON articles USING gin(tsv);
CREATE INDEX articles_embedding_idx ON articles USING hnsw (embedding vector_cosine_ops);

-- Combine keyword and semantic search using Reciprocal Rank Fusion (RRF)
WITH keyword_results AS (
    SELECT id, ts_rank(tsv, plainto_tsquery('english', :query)) AS keyword_rank
    FROM articles
    WHERE tsv @@ plainto_tsquery('english', :query)
    ORDER BY keyword_rank DESC
    LIMIT 20
),
vector_results AS (
    SELECT id, embedding <=> :query_embedding AS vector_distance
    FROM articles
    ORDER BY embedding <=> :query_embedding
    LIMIT 20
),
combined AS (
    SELECT COALESCE(k.id, v.id) AS id,
           COALESCE(1.0 / (60 + ROW_NUMBER() OVER (ORDER BY k.keyword_rank DESC NULLS LAST)), 0) +
           COALESCE(1.0 / (60 + ROW_NUMBER() OVER (ORDER BY v.vector_distance ASC NULLS LAST)), 0) AS rrf_score
    FROM keyword_results k
    FULL OUTER JOIN vector_results v ON k.id = v.id
)
SELECT a.id, a.title, c.rrf_score
FROM combined c
JOIN articles a ON a.id = c.id
ORDER BY c.rrf_score DESC
LIMIT 10;
```

### Distance Threshold Query

```sql
-- Find all vectors within a cosine distance threshold
SELECT id, content, embedding <=> :query_embedding AS distance
FROM documents
WHERE embedding <=> :query_embedding < 0.3
ORDER BY embedding <=> :query_embedding;
```

## Limitations and Considerations

- **vector(n) dimension limit**: Dense vectors are limited to 2,000 dimensions. Models producing higher-dimensional embeddings require dimensionality reduction or use of `halfvec` (up to 4,000 dimensions).
- **Approximate recall**: HNSW and IVFFlat indexes provide approximate results. Recall depends on index parameters and may not reach 100% without exact (sequential) scan.
- **Index build time**: HNSW index construction can be slow on large datasets (millions of vectors). IVFFlat builds faster but requires the table to already contain data for effective clustering.
- **Memory consumption**: HNSW indexes reside in memory and can be substantial for large datasets. Sizing depends on the number of vectors, dimensionality, and the `m` parameter.
- **No built-in sharding**: pgvector relies on PostgreSQL's native partitioning and external sharding solutions (such as Citus) for horizontal scaling. It does not provide built-in distributed vector search.
- **IVFFlat requires pre-populated data**: Building an IVFFlat index on an empty or very small table produces poor clusters. The table should contain a representative sample of data before index creation.
- **Single-node scaling**: Performance is bounded by single-node PostgreSQL limits. For datasets exceeding hundreds of millions of vectors, purpose-built vector databases may offer better throughput.

## Changelog Highlights

- **v0.8.0**: Added iterative scan modes (`strict_order`, `relaxed_order`) for improved filtered vector search. Added `sparsevec` type.
- **v0.7.0**: Added `halfvec` type for half-precision vectors. Added L1 distance operator (`<+>`).
- **v0.6.0**: Added HNSW index support. Previously only IVFFlat was available.
- **v0.5.0**: Added parallel index builds. Added `bit` type with Hamming and Jaccard distance operators.

## Citations

- [1] pgvector GitHub repository: https://github.com/pgvector/pgvector
