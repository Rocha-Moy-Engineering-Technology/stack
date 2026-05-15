[Header 1 ("pgvector", [], []) [Str "pgvector"], BlockQuote [Para [Str "PostgreSQL extension for vector similarity search and embeddings storage"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "pgvector"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Vector Databases"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "pgvector/pgvector"] ("https://github.com/pgvector/pgvector", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "21284"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "github.com/pgvector/pgvector"] ("https://github.com/pgvector/pgvector", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "pgvector is an open-source PostgreSQL extension that adds vector similarity search capabilities directly into PostgreSQL. It supports exact and approximate nearest neighbor search with single-precision, half-precision, binary, and sparse vector types. Rather than requiring a separate vector database, pgvector stores embedding vectors alongside relational data in the same database, leveraging PostgreSQL's ACID transactions, JOINs, indexing, query planning, Write-Ahead Log (WAL) replication, and point-in-time recovery. Written in C, it runs as a native in-process extension inside the PostgreSQL backend, avoiding network round-trips to external services. The extension requires PostgreSQL 13 or later and is available through Docker, Homebrew, APT, Yum, pkg, APK, conda-forge, PGXN, Postgres.app, and many managed cloud providers including Amazon RDS, Azure Database for PostgreSQL, Google Cloud SQL, Heroku Postgres, and Supabase."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("vector-data-types", ["unnumbered", "unlisted"], []) [Str "Vector Data Types"], Para [Str "pgvector introduces four data types for different vector representations:"], BulletList [[Plain [Strong [Str "vector(n)"], Str ": Dense single-precision (32-bit float) vectors supporting up to 2,000 dimensions for indexed columns and up to 16,000 dimensions for unindexed storage. This is the primary type for embeddings from models such as OpenAI, Cohere, or sentence-transformers."]], [Plain [Strong [Str "halfvec(n)"], Str ": Half-precision (16-bit float) vectors supporting up to 4,000 dimensions. Reduces storage by half while maintaining reasonable accuracy for most retrieval tasks."]], [Plain [Strong [Str "bit(n)"], Str ": Binary vectors supporting up to 64,000 dimensions. Each dimension is a single bit, suitable for binary quantization schemes."]], [Plain [Strong [Str "sparsevec(n)"], Str ": Sparse vectors supporting up to 1,000 non-zero elements. Efficient for high-dimensional vectors where most values are zero, such as TF-IDF or BM25 representations. Format: ", Code ("", [], []) "'{0:0.1,10:0.2}'::sparsevec", Str "."]]], Para [Str "All vectors in a column must have matching dimensions; mixed-dimension columns are not supported."], Header 3 ("distance-functions", ["unnumbered", "unlisted"], []) [Str "Distance Functions"], Para [Str "pgvector provides six distance operators, all returning lower values for greater similarity:"], BulletList [[Plain [Strong [Code ("", [], []) "<->", Str " (L2 distance)"], Str ": Euclidean distance. Best general-purpose metric when vectors are not normalized."]], [Plain [Strong [Code ("", [], []) "<#>", Str " (negative inner product)"], Str ": Returns the negative dot product. Multiply by -1 to get actual inner product. Useful when vector magnitude carries meaningful signal."]], [Plain [Strong [Code ("", [], []) "<=>", Str " (cosine distance)"], Str ": Measures the angle between vectors, ignoring magnitude. Subtract from 1 to get cosine similarity. Preferred when vectors are normalized or magnitude should not influence results."]], [Plain [Strong [Code ("", [], []) "<+>", Str " (L1 distance)"], Str ": Manhattan distance. Sum of absolute differences across dimensions."]], [Plain [Strong [Code ("", [], []) "<~>", Str " (Hamming distance)"], Str ": Counts positions where corresponding bits differ. Operates on ", Code ("", [], []) "bit", Str " type only."]], [Plain [Strong [Code ("", [], []) "<%>", Str " (Jaccard distance)"], Str ": Measures dissimilarity between bit sets. Operates on ", Code ("", [], []) "bit", Str " type only."]]], Header 3 ("indexing-strategies", ["unnumbered", "unlisted"], []) [Str "Indexing Strategies"], Para [Str "Without an index, pgvector performs exact nearest neighbor search by scanning all rows, guaranteeing perfect recall but not scaling beyond small datasets. Two approximate nearest neighbor (ANN) index types trade recall for speed:"], BulletList [[Plain [Strong [Str "HNSW (Hierarchical Navigable Small World)"], Str ": A graph-based index implementing a multilayer navigable small world structure. Provides better query performance (higher recall at given latency) than IVFFlat but has slower build times and higher memory usage. No training step required -- indexes can be created on empty tables. The implementation follows the original HNSW paper algorithms for search, neighbor selection, and element insertion."]], [Plain [Strong [Str "IVFFlat (Inverted File with Flat compression)"], Str ": A partition-based index that clusters vectors into lists using k-means, then searches the closest cluster subsets. Faster to build and uses less memory than HNSW but generally provides lower recall. Requires the table to already contain representative data before index creation for effective clustering."]]], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "pgvector operates as a native PostgreSQL extension running within the PostgreSQL backend process:"], BulletList [[Plain [Strong [Str "In-process execution"], Str ": Vector operations run inside the PostgreSQL backend, avoiding network round-trips to external services. Distance calculations are implemented in C with CPU-dispatched SIMD optimizations on Linux x86-64."]], [Plain [Strong [Str "WAL integration"], Str ": All vector data and index changes are written to the Write-Ahead Log, ensuring crash recovery and streaming replication work identically to standard PostgreSQL tables."]], [Plain [Strong [Str "Planner integration"], Str ": The PostgreSQL query planner can combine vector index scans with B-tree index scans, filters, and joins in a single query plan. Cost estimation is tuned to help the planner select between sequential scan, vector index scan, and exact index scan depending on filter selectivity."]], [Plain [Strong [Str "Shared buffer usage"], Str ": Vector data and indexes use PostgreSQL's shared buffer pool and benefit from the same caching, memory management, and vacuum processes as regular tables."]], [Plain [Strong [Str "HNSW internals"], Str ": The HNSW implementation uses a multilayer graph structure with per-element neighbor arrays at each layer. Elements are stored as heap tuples with TID-based visited tracking for on-disk scans. The code implements the original paper's algorithms for search layer traversal (Algorithm 2), neighbor selection with pruning (Algorithm 4), and element insertion (Algorithm 1)."]], [Plain [Strong [Str "Storage"], Str ": Vector data uses ", Code ("", [], []) "external", Str " storage (stored out-of-line, not compressed), preventing TOAST compression overhead on vector columns."]]], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Colocation of vectors and relational data"], Str ": Store embeddings in the same table as metadata, foreign keys, and other columns without external synchronization."]], [Plain [Strong [Str "ACID transactions"], Str ": Vector inserts, updates, and deletes participate in PostgreSQL transactions with full consistency guarantees during concurrent writes."]], [Plain [Strong [Str "JOIN support"], Str ": Combine vector similarity search with relational joins in a single query."]], [Plain [Strong [Str "Six distance metrics"], Str ": L2, cosine, inner product, L1, Hamming, and Jaccard distances with corresponding operators and index support."]], [Plain [Strong [Str "Two ANN index types"], Str ": HNSW for higher recall and IVFFlat for faster builds with lower memory consumption."]], [Plain [Strong [Str "Iterative index scans (v0.8.0+)"], Str ": Automatically scan more of the index when filters reduce result count, available in ", Code ("", [], []) "strict_order", Str " mode for both HNSW and IVFFlat."]], [Plain [Strong [Str "Half-precision vectors"], Str ": ", Code ("", [], []) "halfvec", Str " type halves storage while supporting all distance functions."]], [Plain [Strong [Str "Sparse vector support"], Str ": ", Code ("", [], []) "sparsevec", Str " type for efficient storage of vectors with mostly zero values."]], [Plain [Strong [Str "Binary quantization"], Str ": ", Code ("", [], []) "binary_quantize()", Str " function converts full-precision vectors to binary for compact storage and fast Hamming distance comparison."]], [Plain [Strong [Str "Subvector indexing"], Str ": Create indexes on vector slices using expression indexes."]], [Plain [Strong [Str "Parallel index builds"], Str ": Both HNSW and IVFFlat support parallel construction workers for faster index creation on multi-core systems."]], [Plain [Strong [Str "Bulk loading"], Str ": COPY with binary format for high-throughput vector insertion."]], [Plain [Strong [Str "Concatenation operator"], Str ": Combine vectors with the concatenation operator."]], [Plain [Strong [Str "Aggregate functions"], Str ": ", Code ("", [], []) "AVG()", Str " and ", Code ("", [], []) "SUM()", Str " for vector columns."]], [Plain [Strong [Str "Helper functions"], Str ": ", Code ("", [], []) "vector_dims()", Str ", ", Code ("", [], []) "vector_norm()", Str ", ", Code ("", [], []) "cosine_similarity()", Str ", ", Code ("", [], []) "l2_normalize()", Str ", ", Code ("", [], []) "subvector()", Str ", ", Code ("", [], []) "hamming_distance()", Str ", ", Code ("", [], []) "jaccard_distance()", Str "."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Retrieval-Augmented Generation (RAG)"], Str ": Store document chunk embeddings and retrieve the most relevant chunks for a query embedding before passing them to a large language model."]], [Plain [Strong [Str "Semantic search"], Str ": Find documents, products, or records by meaning rather than keyword matching."]], [Plain [Strong [Str "Hybrid search"], Str ": Combine full-text search (", Code ("", [], []) "tsvector", Str "/", Code ("", [], []) "tsquery", Str ") with vector similarity search in a single PostgreSQL query using reciprocal rank fusion or other combination strategies."]], [Plain [Strong [Str "Recommendation systems"], Str ": Compute similarity between user and item embeddings stored alongside transactional data."]], [Plain [Strong [Str "Duplicate detection"], Str ": Identify near-duplicate records by finding vectors within a small distance threshold."]], [Plain [Strong [Str "Image retrieval"], Str ": Store image embeddings and find visually similar images using cosine or L2 distance."]], [Plain [Strong [Str "Classification"], Str ": Use nearest neighbor vectors with known labels to classify new items."]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("table-definition", ["unnumbered", "unlisted"], []) [Str "Table Definition"], CodeBlock ("", ["sql"], []) "CREATE TABLE documents (
    id BIGSERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(1536)
);

-- Add vector column to existing table
ALTER TABLE documents ADD COLUMN embedding vector(1536);
", Header 3 ("insert-vectors", ["unnumbered", "unlisted"], []) [Str "Insert Vectors"], CodeBlock ("", ["sql"], []) "-- Single insert
INSERT INTO documents (content, embedding)
VALUES ('Sample text', '[0.1, 0.2, 0.3, ...]');

-- Bulk upsert
INSERT INTO documents (id, content, embedding)
VALUES (1, 'text one', '[0.1, 0.2, 0.3]'), (2, 'text two', '[0.4, 0.5, 0.6]')
ON CONFLICT (id) DO UPDATE SET embedding = EXCLUDED.embedding;
", Header 3 ("nearest-neighbor-queries", ["unnumbered", "unlisted"], []) [Str "Nearest Neighbor Queries"], CodeBlock ("", ["sql"], []) "-- L2 distance (Euclidean)
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
", Header 3 ("create-indexes", ["unnumbered", "unlisted"], []) [Str "Create Indexes"], CodeBlock ("", ["sql"], []) "-- HNSW indexes
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
", Header 3 ("index-operator-classes", ["unnumbered", "unlisted"], []) [Str "Index Operator Classes"], Para [Str "HNSW supported types: ", Code ("", [], []) "vector", Str " (up to 2,000 dimensions), ", Code ("", [], []) "halfvec", Str " (up to 4,000 dimensions), ", Code ("", [], []) "bit", Str " (up to 64,000 dimensions), ", Code ("", [], []) "sparsevec", Str " (up to 1,000 non-zero elements)."], Para [Str "IVFFlat supported types: ", Code ("", [], []) "vector", Str " (up to 2,000 dimensions), ", Code ("", [], []) "halfvec", Str " (up to 4,000 dimensions), ", Code ("", [], []) "bit", Str " (up to 64,000 dimensions)."], Para [Str "Operator classes by distance function:"], BulletList [[Plain [Str "L2: ", Code ("", [], []) "vector_l2_ops", Str ", ", Code ("", [], []) "halfvec_l2_ops"]], [Plain [Str "Inner product: ", Code ("", [], []) "vector_ip_ops", Str ", ", Code ("", [], []) "halfvec_ip_ops"]], [Plain [Str "Cosine: ", Code ("", [], []) "vector_cosine_ops", Str ", ", Code ("", [], []) "halfvec_cosine_ops"]], [Plain [Str "L1: ", Code ("", [], []) "vector_l1_ops", Str ", ", Code ("", [], []) "halfvec_l1_ops"]], [Plain [Str "Hamming: ", Code ("", [], []) "bit_hamming_ops"]], [Plain [Str "Jaccard: ", Code ("", [], []) "bit_jaccard_ops"]]], Header 3 ("half-precision-vectors", ["unnumbered", "unlisted"], []) [Str "Half-Precision Vectors"], CodeBlock ("", ["sql"], []) "CREATE TABLE items (id bigserial PRIMARY KEY, embedding halfvec(3));
INSERT INTO items (embedding) VALUES ('[0.1, 0.2, 0.3]');
CREATE INDEX ON items USING hnsw (embedding halfvec_cosine_ops);
", Header 3 ("binary-vectors", ["unnumbered", "unlisted"], []) [Str "Binary Vectors"], CodeBlock ("", ["sql"], []) "CREATE TABLE items (id bigserial PRIMARY KEY, embedding bit(8));
INSERT INTO items (embedding) VALUES ('10101010'), ('11001100');

-- Binary quantization from full-precision vectors
SELECT binary_quantize(embedding) FROM documents;
", Header 3 ("sparse-vectors", ["unnumbered", "unlisted"], []) [Str "Sparse Vectors"], CodeBlock ("", ["sql"], []) "CREATE TABLE items (id bigserial PRIMARY KEY, embedding sparsevec(1000));
INSERT INTO items (embedding) VALUES ('{0:0.1,10:0.2}'::sparsevec);
", Header 3 ("aggregate-and-helper-functions", ["unnumbered", "unlisted"], []) [Str "Aggregate and Helper Functions"], CodeBlock ("", ["sql"], []) "-- Aggregates
SELECT AVG(embedding) FROM documents;
SELECT SUM(embedding) FROM documents;
SELECT category_id, AVG(embedding) FROM documents GROUP BY category_id;

-- Helper functions
SELECT vector_dims(embedding) FROM documents LIMIT 1;
SELECT vector_norm(embedding) FROM documents LIMIT 1;
SELECT cosine_similarity(a.embedding, b.embedding) FROM documents a, documents b WHERE a.id = 1 AND b.id = 2;
SELECT l2_normalize(embedding) FROM documents LIMIT 1;
SELECT subvector(embedding, 1, 100) FROM documents LIMIT 1;
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("hnsw-parameters", ["unnumbered", "unlisted"], []) [Str "HNSW Parameters"], Para [Str "Index build parameters (set at creation time):"], BulletList [[Plain [Strong [Code ("", [], []) "m"], Str " (default 16): Maximum number of connections per node in each layer. Higher values improve recall but increase memory and build time."]], [Plain [Strong [Code ("", [], []) "ef_construction"], Str " (default 64): Size of the dynamic candidate list during index construction. Higher values improve recall at the cost of slower builds."]]], Para [Str "Query-time parameters (set per session or transaction):"], BulletList [[Plain [Strong [Code ("", [], []) "hnsw.ef_search"], Str " (default 40): Size of the dynamic candidate list during search. Higher values improve recall at the cost of higher latency."]], [Plain [Strong [Code ("", [], []) "hnsw.iterative_scan"], Str " (default ", Code ("", [], []) "off", Str "): Enable iterative scans with ", Code ("", [], []) "strict_order", Str " to automatically scan more of the index when filters reduce result count."]]], CodeBlock ("", ["sql"], []) "SET hnsw.ef_search = 100;
SET hnsw.iterative_scan = strict_order;

-- Transaction-scoped setting
BEGIN;
SET LOCAL hnsw.ef_search = 200;
SELECT ...;
COMMIT;
", Header 3 ("ivfflat-parameters", ["unnumbered", "unlisted"], []) [Str "IVFFlat Parameters"], Para [Str "Index build parameter:"], BulletList [[Plain [Strong [Code ("", [], []) "lists"], Str ": Number of inverted lists (clusters). Starting point: ", Code ("", [], []) "rows / 1000", Str " for up to 1M rows, ", Code ("", [], []) "sqrt(rows)", Str " for over 1M rows."]]], Para [Str "Query-time parameters:"], BulletList [[Plain [Strong [Code ("", [], []) "ivfflat.probes"], Str " (default 1): Number of lists to search. Starting point: ", Code ("", [], []) "sqrt(lists)", Str ". Setting to the total number of lists produces exact search (planner will not use the index)."]], [Plain [Strong [Code ("", [], []) "ivfflat.iterative_scan"], Str " (default ", Code ("", [], []) "off", Str "): Enable with ", Code ("", [], []) "strict_order", Str " for automatic additional scanning."]]], CodeBlock ("", ["sql"], []) "SET ivfflat.probes = 10;
SET ivfflat.iterative_scan = strict_order;
", Header 3 ("index-build-performance", ["unnumbered", "unlisted"], []) [Str "Index Build Performance"], CodeBlock ("", ["sql"], []) "-- Increase work memory for faster HNSW builds (graph must fit in memory)
SET maintenance_work_mem = '8GB';

-- Parallelize index construction (both HNSW and IVFFlat)
SET max_parallel_maintenance_workers = 7;  -- plus leader

-- May also need to increase max_parallel_workers (default: 8)
SET max_parallel_workers = 15;
", Para [Str "A notice appears when the HNSW graph no longer fits in ", Code ("", [], []) "maintenance_work_mem", Str ":"], CodeBlock ("", [""], []) "NOTICE: hnsw graph no longer fits into maintenance_work_mem after 100000 tuples
DETAIL: Building will take significantly more time.
HINT: Increase maintenance_work_mem to speed up builds.
", Header 3 ("index-build-progress-monitoring", ["unnumbered", "unlisted"], []) [Str "Index Build Progress Monitoring"], CodeBlock ("", ["sql"], []) "-- Check progress during index creation
SELECT phase, round(100.0 * blocks_done / nullif(blocks_total, 0), 1) AS \"%\"
FROM pg_stat_progress_create_index;
", Para [Str "HNSW phases: ", Code ("", [], []) "initializing", Str ", ", Code ("", [], []) "loading tuples", Str "."], Para [Str "IVFFlat phases: ", Code ("", [], []) "initializing", Str ", ", Code ("", [], []) "performing k-means", Str ", ", Code ("", [], []) "assigning tuples", Str ", ", Code ("", [], []) "loading tuples", Str " (percentage only populated during ", Code ("", [], []) "loading tuples", Str ")."], Header 3 ("general-recommendations", ["unnumbered", "unlisted"], []) [Str "General Recommendations"], BulletList [[Plain [Str "Create indexes after loading initial data for better performance."]], [Plain [Str "For HNSW, increase ", Code ("", [], []) "hnsw.ef_search", Str " to improve recall at the cost of higher latency."]], [Plain [Str "For IVFFlat, increase ", Code ("", [], []) "ivfflat.probes", Str " to improve recall; use ", Code ("", [], []) "sqrt(lists)", Str " as a starting point."]], [Plain [Str "Set ", Code ("", [], []) "maintenance_work_mem", Str " to at least 1-8 GB when building indexes on large tables."]], [Plain [Str "Use ", Code ("", [], []) "max_parallel_maintenance_workers", Str " to speed up index creation on multi-core systems."]], [Plain [Str "Use COPY with binary format for bulk data loading."]], [Plain [Str "For filtered queries, create B-tree indexes on filter columns; enable iterative scans for approximate indexes with filters."]], [Plain [Str "For few distinct filter values, use partial indexes; for many values, use table partitioning."]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("python-with-psycopg3", ["unnumbered", "unlisted"], []) [Str "Python with psycopg3"], CodeBlock ("", ["python"], []) "import psycopg

conn = psycopg.connect(\"dbname=mydb\")
conn.execute(\"CREATE EXTENSION IF NOT EXISTS vector\")
conn.execute(\"\"\"
    CREATE TABLE IF NOT EXISTS documents (
        id BIGSERIAL PRIMARY KEY,
        content TEXT,
        embedding vector(1536)
    )
\"\"\")

# Insert
embedding = [0.1, 0.2, 0.3]  # truncated
conn.execute(
    \"INSERT INTO documents (content, embedding) VALUES (%s, %s)\",
    (\"sample text\", str(embedding))
)

# Query nearest neighbors
query_embedding = [0.1, 0.2, 0.3]
results = conn.execute(
    \"SELECT id, content FROM documents ORDER BY embedding <=> %s LIMIT 5\",
    (str(query_embedding),)
).fetchall()
", Header 3 ("python-with-pgvector-python", ["unnumbered", "unlisted"], []) [Str "Python with pgvector-python"], Para [Str "The ", Code ("", [], []) "pgvector", Str " Python package (", Code ("", [], []) "pip install pgvector", Str ") provides native type support for multiple drivers:"], CodeBlock ("", ["python"], []) "from pgvector.psycopg import register_vector
import psycopg
import numpy as np

conn = psycopg.connect(\"dbname=mydb\")
register_vector(conn)

# Insert with numpy array (no manual string conversion)
embedding = np.array([0.1, 0.2, 0.3])
conn.execute(
    \"INSERT INTO documents (content, embedding) VALUES (%s, %s)\",
    (\"sample text\", embedding)
)
", Para [Str "Supported drivers: psycopg3, psycopg2, asyncpg, pg8000."], Header 3 ("sqlalchemy", ["unnumbered", "unlisted"], []) [Str "SQLAlchemy"], CodeBlock ("", ["python"], []) "from pgvector.sqlalchemy import Vector
from sqlalchemy import Column, Integer, Text, create_engine
from sqlalchemy.orm import declarative_base, Session

Base = declarative_base()

class Document(Base):
    __tablename__ = \"documents\"
    id = Column(Integer, primary_key=True)
    content = Column(Text)
    embedding = Column(Vector(1536))

engine = create_engine(\"postgresql://user:password@localhost/mydb\")
Base.metadata.create_all(engine)

with Session(engine) as session:
    session.add(Document(content=\"text\", embedding=[0.1, 0.2, 0.3]))
    session.commit()

    # Nearest neighbor query
    from pgvector.sqlalchemy import Vector
    results = session.query(Document).order_by(
        Document.embedding.cosine_distance([0.1, 0.2, 0.3])
    ).limit(5).all()
", Header 3 ("django", ["unnumbered", "unlisted"], []) [Str "Django"], CodeBlock ("", ["python"], []) "from pgvector.django import VectorExtension, VectorField, HnswIndex

# Migration
class Migration(migrations.Migration):
    operations = [VectorExtension()]

# Model
class Document(models.Model):
    content = models.TextField()
    embedding = VectorField(dimensions=1536)

    class Meta:
        indexes = [HnswIndex(fields=['embedding'], opclasses=['vector_cosine_ops'])]
", Header 3 ("langchain", ["unnumbered", "unlisted"], []) [Str "LangChain"], CodeBlock ("", ["python"], []) "from langchain_community.vectorstores import PGVector

connection_string = \"postgresql://user:password@localhost:5432/mydb\"
vectorstore = PGVector.from_documents(
    documents=docs,
    embedding=embeddings_model,
    connection_string=connection_string,
    collection_name=\"my_collection\",
)
results = vectorstore.similarity_search(\"query text\", k=5)
", Header 3 ("other-language-libraries", ["unnumbered", "unlisted"], []) [Str "Other Language Libraries"], Para [Str "pgvector works with any language that has a PostgreSQL client. Official client libraries exist for Ruby, JavaScript/TypeScript (node-postgres, Knex.js, Objection.js, Sequelize, Prisma), PHP (Laravel), Go, Java (JDBC, Spring), Rust, .NET (Npgsql, Entity Framework Core), Elixir (Ecto), and Lua."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-rag-pipeline", ["unnumbered", "unlisted"], []) [Str "Basic RAG Pipeline"], CodeBlock ("", ["sql"], []) "CREATE TABLE chunks (
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
", Header 3 ("hybrid-search-with-full-text-and-vector", ["unnumbered", "unlisted"], []) [Str "Hybrid Search with Full-Text and Vector"], CodeBlock ("", ["sql"], []) "CREATE TABLE articles (
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
", Header 3 ("binary-quantization-with-re-ranking", ["unnumbered", "unlisted"], []) [Str "Binary Quantization with Re-ranking"], CodeBlock ("", ["sql"], []) "-- Create binary quantized column for fast initial retrieval
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
", Header 3 ("filtered-vector-search-with-iterative-scans", ["unnumbered", "unlisted"], []) [Str "Filtered Vector Search with Iterative Scans"], CodeBlock ("", ["sql"], []) "-- Enable iterative scans for better filtered search
SET hnsw.iterative_scan = strict_order;

SELECT id, content
FROM documents
WHERE category = 'technical' AND created_at > '2025-01-01'
ORDER BY embedding <=> :query_embedding
LIMIT 10;
", Header 3 ("partitioned-table-for-scaling", ["unnumbered", "unlisted"], []) [Str "Partitioned Table for Scaling"], CodeBlock ("", ["sql"], []) "CREATE TABLE items (
    id BIGSERIAL,
    embedding vector(1536),
    category_id INT
) PARTITION BY LIST(category_id);

CREATE TABLE items_cat_1 PARTITION OF items FOR VALUES IN (1);
CREATE TABLE items_cat_2 PARTITION OF items FOR VALUES IN (2);

-- Create per-partition indexes
CREATE INDEX ON items_cat_1 USING hnsw (embedding vector_cosine_ops);
CREATE INDEX ON items_cat_2 USING hnsw (embedding vector_cosine_ops);
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "vector(n) dimension limit"], Str ": Dense vector indexes are limited to 2,000 dimensions. Unindexed vector columns support up to 16,000 dimensions. Models producing higher-dimensional embeddings require dimensionality reduction or ", Code ("", [], []) "halfvec", Str " (up to 4,000 indexed dimensions)."]], [Plain [Strong [Str "Approximate recall"], Str ": HNSW and IVFFlat indexes provide approximate results. Recall depends on index parameters and may not reach 100% without exact (sequential) scan."]], [Plain [Strong [Str "HNSW build time"], Str ": HNSW index construction can be slow on large datasets (millions of vectors) and requires the graph to fit in ", Code ("", [], []) "maintenance_work_mem", Str " for optimal build speed."]], [Plain [Strong [Str "HNSW memory consumption"], Str ": HNSW indexes reside in memory and can be substantial. Sizing depends on number of vectors, dimensionality, and the ", Code ("", [], []) "m", Str " parameter. Indexes can exceed memory but performance degrades with disk access."]], [Plain [Strong [Str "IVFFlat requires pre-populated data"], Str ": Building an IVFFlat index on an empty or very small table produces poor clusters. The table should contain a representative sample of data before index creation."]], [Plain [Strong [Str "No built-in sharding"], Str ": pgvector relies on PostgreSQL's native partitioning and external sharding solutions (such as Citus) for horizontal scaling. It does not provide built-in distributed vector search."]], [Plain [Strong [Str "Single-node scaling"], Str ": Performance is bounded by single-node PostgreSQL limits. For datasets exceeding hundreds of millions of vectors, purpose-built distributed vector databases may offer better throughput."]], [Plain [Strong [Str "Fixed dimensions per column"], Str ": All vectors in a column must have the same number of dimensions. Mixed-dimension storage requires separate columns or tables."]], [Plain [Strong [Str "sparsevec index support"], Str ": Sparse vectors support HNSW indexing only (not IVFFlat) and are limited to L2, inner product, and cosine distances."]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "v0.8.2"], Str " (2026-02-25): Fixed buffer overflow with parallel HNSW index build. Improved Windows install target. Fixed EXPLAIN output for Postgres 18."]], [Plain [Strong [Str "v0.8.1"], Str " (2025-09-04): Added support for Postgres 18 rc1. Improved ", Code ("", [], []) "binary_quantize", Str " performance."]], [Plain [Strong [Str "v0.8.0"], Str " (2024-10-30): Added iterative index scans for both HNSW and IVFFlat. Added array-to-sparsevec casts. Improved cost estimation for filtered queries. Improved HNSW scan, insert, and on-disk build performance. Dropped Postgres 12 support."]], [Plain [Strong [Str "v0.7.0"], Str " (2024-04-29): Added ", Code ("", [], []) "halfvec", Str " and ", Code ("", [], []) "sparsevec", Str " types. Added ", Code ("", [], []) "bit", Str " type indexing. Added L1 distance indexing for HNSW. Added ", Code ("", [], []) "binary_quantize", Str ", ", Code ("", [], []) "hamming_distance", Str ", ", Code ("", [], []) "jaccard_distance", Str ", ", Code ("", [], []) "l2_normalize", Str ", ", Code ("", [], []) "subvector", Str " functions. Added vector concatenation operator. Added CPU dispatching for distance functions on Linux x86-64."]], [Plain [Strong [Str "v0.6.0"], Str " (2024-01-29): Added parallel HNSW index builds. Changed vector storage from ", Code ("", [], []) "extended", Str " to ", Code ("", [], []) "external", Str ". Improved HNSW performance and reduced WAL generation. Moved Docker image to ", Code ("", [], []) "pgvector", Str " org. Dropped Postgres 11 support."]], [Plain [Strong [Str "v0.5.0"], Str " (2023-08-28): Added HNSW index type. Added parallel IVFFlat builds. Added ", Code ("", [], []) "l1_distance", Str " function, element-wise multiplication, ", Code ("", [], []) "sum", Str " aggregate. Improved distance function performance."]], [Plain [Strong [Str "v0.4.0"], Str " (2023-01-11): Increased max vector dimensions from 1,024 to 16,000 (indexed: 2,000). Changed storage from ", Code ("", [], []) "plain", Str " to ", Code ("", [], []) "extended", Str ". Added ", Code ("", [], []) "avg", Str " aggregate. Added experimental Windows support. Dropped Postgres 10 support."]], [Plain [Strong [Str "v0.3.0"], Str " (2022-10-15): Added Postgres 15 support. Dropped Postgres 9.6 support."]], [Plain [Strong [Str "v0.1.0"], Str " (2021-04-20): First release with IVFFlat indexing, L2/inner product/cosine distance operators, and basic vector type."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " pgvector GitHub repository and README - https://github.com/pgvector/pgvector"]], [Plain [Str "[", Str "2", Str "]", Str " pgvector Changelog - https://github.com/pgvector/pgvector/blob/master/CHANGELOG.md"]], [Plain [Str "[", Str "3", Str "]", Str " pgvector-python client library - https://github.com/pgvector/pgvector-python"]], [Plain [Str "[", Str "4", Str "]", Str " pgvector HNSW implementation (hnswutils.c) - https://github.com/pgvector/pgvector/blob/master/src/hnswutils.c"]], [Plain [Str "[", Str "5", Str "]", Str " pgvector GitHub API metadata - https://api.github.com/repos/pgvector/pgvector"]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], Para [Str "pgvector, PostgreSQL extension, vector similarity search, vector(n), halfvec, sparsevec, bit, HNSW, IVFFlat, L2 distance, cosine distance, inner product, L1 distance, Hamming distance, Jaccard distance, ACID transactions, JOIN, WAL replication, iterative scans, binary_quantize, hnsw.ef_search, ivfflat.probes, vector_cosine_ops, halfvec_cosine_ops, RDS, Supabase, Azure Database for PostgreSQL, Cloud SQL, Heroku Postgres"], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Install the extension with ", Code ("", [], []) "CREATE EXTENSION vector"]], [Plain [Str "Create a ", Code ("", [], []) "vector(1536)", Str " column on an existing table"]], [Plain [Str "Query nearest neighbors with ", Code ("", [], []) "<->", Str ", ", Code ("", [], []) "<=>", Str ", ", Code ("", [], []) "<#>", Str ", ", Code ("", [], []) "<+>", Str ", ", Code ("", [], []) "<~>", Str ", ", Code ("", [], []) "<%>"]], [Plain [Str "Build an HNSW index with ", Code ("", [], []) "vector_cosine_ops"]], [Plain [Str "Build an IVFFlat index with chosen ", Code ("", [], []) "lists"]], [Plain [Str "Tune ", Code ("", [], []) "hnsw.ef_search", Str " and ", Code ("", [], []) "ivfflat.probes", Str " per session"]], [Plain [Str "Enable iterative scans for filtered queries (", Code ("", [], []) "strict_order", Str ")"]], [Plain [Str "Use ", Code ("", [], []) "halfvec", Str " for half-precision storage (up to 4,000 indexed dims)"]], [Plain [Str "Use ", Code ("", [], []) "sparsevec", Str " for TF-IDF / BM25-style sparse vectors"]], [Plain [Str "Convert dense vectors to binary with ", Code ("", [], []) "binary_quantize"]], [Plain [Str "Combine ", Code ("", [], []) "tsvector", Str " full-text and vector search via RRF"]], [Plain [Str "Build subvector or partial indexes for filtered RAG"]], [Plain [Str "Parallelize index builds with ", Code ("", [], []) "max_parallel_maintenance_workers"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "How do I add vector search to my existing PostgreSQL database?"]], [Plain [Str "How do I join embeddings with relational metadata in one query?"]], [Plain [Str "How do I do hybrid full-text + vector search inside Postgres?"]], [Plain [Str "Do I get ACID transactions with my embedding inserts?"]], [Plain [Str "Can I run vector search on Amazon RDS or Supabase?"]], [Plain [Str "How do I pick between HNSW and IVFFlat for my dataset?"]], [Plain [Str "How much memory does an HNSW index need?"]], [Plain [Str "How do I quantize vectors for storage efficiency?"]], [Plain [Str "How do I scale beyond a single PostgreSQL node?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Adding a separate vector database doubles my infrastructure and synchronization cost"]], [Plain [Str "I lose ACID transactions when vectors live outside my main database"]], [Plain [Str "I cannot JOIN vector results with my relational data efficiently"]], [Plain [Str "Filtered vector search is slow without iterative scans"]], [Plain [Str "HNSW indexes consume large amounts of RAM"]], [Plain [Str "Single-node PostgreSQL caps at hundreds of millions of vectors"]], [Plain [Str "High-dimensional embeddings exceed the indexed limit (2,000 dims for ", Code ("", [], []) "vector", Str ")"]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when you already run PostgreSQL and want vectors next to relational data with ACID, JOINs, and WAL replication"]], [Plain [Str "Pick this over Pinecone when you want self-hosted, transactional, and no extra infrastructure"]], [Plain [Str "Pick this over Weaviate / Qdrant / Milvus when single-node Postgres scale is sufficient and operational simplicity wins"]], [Plain [Str "Pick this when managed PostgreSQL (RDS, Cloud SQL, Azure, Supabase, Heroku) is already in your stack"]], [Plain [Str "Pick this when six distance metrics, partial indexes, and SQL-native hybrid (RRF) fit your application"]], [Plain [Str "Skip this when you need billion-scale distributed vector search; reach for Milvus or Qdrant instead"]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "pgvector extension"]], [Plain [Str "pgvector-python"]], [Plain [Str "vector(n) / halfvec / sparsevec / bit"]], [Plain [Str "HNSW / IVFFlat"]], [Plain [Str "Iterative index scan"]], [Plain [Str "Distance operators (", Code ("", [], []) "<->", Str ", ", Code ("", [], []) "<=>", Str ", ", Code ("", [], []) "<#>", Str ", ", Code ("", [], []) "<+>", Str ", ", Code ("", [], []) "<~>", Str ", ", Code ("", [], []) "<%>", Str ")"]], [Plain [Str "vector_cosine_ops"]], [Plain [Str "binary_quantize / Hamming"]], [Plain [Str "Postgres-native vector search"]], [Plain [Str "RDS pgvector / Supabase pgvector / Cloud SQL pgvector"]]]]