[Header 1 ("pgvector", [], []) [Str "pgvector"], BlockQuote [Para [Str "PostgreSQL extension for vector similarity search, enabling storage and retrieval of embeddings alongside relational data with full ACID compliance."]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "RAG & Knowledge Retrieval"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/pgvector/pgvector"] ("https://github.com/pgvector/pgvector", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "19914"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://github.com/pgvector/pgvector", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "pgvector is a PostgreSQL extension that adds vector similarity search capabilities directly into PostgreSQL. Rather than requiring a separate vector database, pgvector allows developers to store embedding vectors alongside relational data in the same database, leveraging PostgreSQL's mature ecosystem of ACID transactions, JOINs, indexing, and query planning. This eliminates the operational overhead of synchronizing data between a relational database and a dedicated vector store, making it a practical choice for applications that need both structured queries and semantic search."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("vector-data-types", ["unnumbered", "unlisted"], []) [Str "Vector Data Types"], Para [Str "pgvector introduces several data types for storing different representations of vectors:"], BulletList [[Plain [Strong [Str "vector(n)"], Str ": Dense floating-point vectors supporting up to 2,000 dimensions. This is the primary type used for storing embeddings from models such as OpenAI, Cohere, or sentence-transformers."]], [Plain [Strong [Str "halfvec(n)"], Str ": Half-precision floating-point vectors supporting up to 4,000 dimensions. Uses 16-bit floats to reduce storage while maintaining reasonable accuracy for many retrieval tasks."]], [Plain [Strong [Str "bit(n)"], Str ": Binary vectors supporting up to 64,000 dimensions. Suitable for binary quantization schemes where each dimension is represented as a single bit."]], [Plain [Strong [Str "sparsevec(n)"], Str ": Sparse vectors supporting up to 1,000 non-zero elements. Efficient for high-dimensional vectors where most values are zero, such as TF-IDF or BM25 representations."]]], Header 3 ("distance-functions", ["unnumbered", "unlisted"], []) [Str "Distance Functions"], Para [Str "pgvector provides six distance operators for computing similarity between vectors:"], BulletList [[Plain [Strong [Code ("", [], []) "<->", Str " (L2 distance)"], Str ": Euclidean distance. Lower values indicate greater similarity. Best general-purpose metric when vectors are not normalized."]], [Plain [Strong [Code ("", [], []) "<#>", Str " (negative inner product)"], Str ": Returns the negative dot product. Lower values indicate greater similarity. Useful when vectors encode magnitude as meaningful signal."]], [Plain [Strong [Code ("", [], []) "<=>", Str " (cosine distance)"], Str ": Measures the angle between vectors, ignoring magnitude. Lower values indicate greater similarity. Preferred when vectors are normalized or when magnitude should not influence results."]], [Plain [Strong [Code ("", [], []) "<+>", Str " (L1 distance)"], Str ": Manhattan distance. Sum of absolute differences across dimensions. Lower values indicate greater similarity."]], [Plain [Strong [Code ("", [], []) "<~>", Str " (Hamming distance)"], Str ": Counts the number of positions where corresponding bits differ. Operates on ", Code ("", [], []) "bit", Str " type vectors."]], [Plain [Strong [Code ("", [], []) "<%>", Str " (Jaccard distance)"], Str ": Measures dissimilarity between bit sets. Operates on ", Code ("", [], []) "bit", Str " type vectors."]]], Header 3 ("indexing-strategies", ["unnumbered", "unlisted"], []) [Str "Indexing Strategies"], Para [Str "pgvector supports two approximate nearest neighbor (ANN) index types:"], BulletList [[Plain [Strong [Str "HNSW (Hierarchical Navigable Small World)"], Str ": A graph-based index that provides better recall at the cost of higher memory usage and slower build times. Default parameters are ", Code ("", [], []) "m=16", Str " (max connections per node) and ", Code ("", [], []) "ef_construction=64", Str " (size of the dynamic candidate list during construction). At query time, ", Code ("", [], []) "hnsw.ef_search=40", Str " controls the search breadth. Increasing ", Code ("", [], []) "ef_search", Str " improves recall but increases latency."]], [Plain [Strong [Str "IVFFlat (Inverted File with Flat compression)"], Str ": A partition-based index that clusters vectors into lists. Faster to build and uses less memory than HNSW but generally provides lower recall. The ", Code ("", [], []) "lists", Str " parameter controls the number of clusters, and ", Code ("", [], []) "ivfflat.probes", Str " controls how many clusters are searched at query time."]]], Para [Str "Without an index, pgvector performs exact nearest neighbor search by scanning all rows, which guarantees perfect recall but does not scale beyond small datasets."], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Header 3 ("from-source", ["unnumbered", "unlisted"], []) [Str "From Source"], CodeBlock ("", ["bash"], []) "git clone --branch v0.8.0 https://github.com/pgvector/pgvector.git
cd pgvector
make
make install
", Para [Str "This requires PostgreSQL development headers (", Code ("", [], []) "postgresql-server-dev-*", Str " on Debian/Ubuntu or the equivalent for your platform)."], Header 3 ("package-managers", ["unnumbered", "unlisted"], []) [Str "Package Managers"], CodeBlock ("", ["bash"], []) "# Homebrew (macOS)
brew install pgvector

# APT (Debian/Ubuntu)
sudo apt install postgresql-17-pgvector

# PGXN
pgxn install vector
", Header 3 ("docker", ["unnumbered", "unlisted"], []) [Str "Docker"], CodeBlock ("", ["dockerfile"], []) "FROM pgvector/pgvector:pg17
", Para [Str "Or add pgvector to an existing PostgreSQL image:"], CodeBlock ("", ["dockerfile"], []) "FROM postgres:17
RUN apt-get update && apt-get install -y postgresql-17-pgvector
", Header 3 ("enabling-the-extension", ["unnumbered", "unlisted"], []) [Str "Enabling the Extension"], Para [Str "After installation, enable pgvector in your database:"], CodeBlock ("", ["sql"], []) "CREATE EXTENSION vector;
", Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "pgvector operates as a native PostgreSQL extension, meaning it runs within the PostgreSQL process and integrates directly with the query planner, executor, and storage engine. Key architectural characteristics include:"], BulletList [[Plain [Strong [Str "In-process execution"], Str ": Vector operations run inside the PostgreSQL backend process, avoiding network round-trips to external services."]], [Plain [Strong [Str "WAL integration"], Str ": All vector data and index changes are written to the Write-Ahead Log (WAL), ensuring crash recovery and replication work identically to standard PostgreSQL tables."]], [Plain [Strong [Str "Planner integration"], Str ": The PostgreSQL query planner can combine vector index scans with other index scans, filters, and joins in a single query plan."]], [Plain [Strong [Str "Shared buffer usage"], Str ": Vector data and indexes use PostgreSQL's shared buffer pool, benefiting from the same caching and memory management as regular tables."]]], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], BulletList [[Plain [Strong [Str "Colocation of vectors and relational data"], Str ": Store embeddings in the same table as metadata, foreign keys, and other columns. No external synchronization required."]], [Plain [Strong [Str "ACID transactions"], Str ": Vector inserts, updates, and deletes participate in PostgreSQL transactions, ensuring consistency even during concurrent writes."]], [Plain [Strong [Str "JOIN support"], Str ": Combine vector similarity search with relational joins, enabling queries such as \"find the most similar documents written by a specific author.\""]], [Plain [Strong [Str "Multiple distance metrics"], Str ": Six built-in distance operators covering Euclidean, cosine, inner product, Manhattan, Hamming, and Jaccard distances."]], [Plain [Strong [Str "Two ANN index types"], Str ": HNSW for higher recall and IVFFlat for faster builds with lower memory consumption."]], [Plain [Strong [Str "Iterative scans (v0.8.0+)"], Str ": ", Code ("", [], []) "strict_order", Str " and ", Code ("", [], []) "relaxed_order", Str " scan modes allow the query planner to interleave index scans with filter evaluation, improving performance when combining vector search with ", Code ("", [], []) "WHERE", Str " clauses."]], [Plain [Strong [Str "Half-precision and sparse vector support"], Str ": Reduce storage with ", Code ("", [], []) "halfvec", Str " or efficiently store sparse representations with ", Code ("", [], []) "sparsevec", Str "."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Retrieval-Augmented Generation (RAG)"], Str ": Store document chunk embeddings and retrieve the most relevant chunks for a given query embedding before passing them to a large language model (LLM)."]], [Plain [Strong [Str "Semantic search"], Str ": Find documents, products, or records by meaning rather than keyword matching."]], [Plain [Strong [Str "Recommendation systems"], Str ": Compute similarity between user and item embeddings stored alongside transactional data."]], [Plain [Strong [Str "Duplicate detection"], Str ": Identify near-duplicate records by finding vectors within a small distance threshold."]], [Plain [Strong [Str "Hybrid search"], Str ": Combine full-text search (", Code ("", [], []) "tsvector", Str ") with vector similarity search in a single PostgreSQL query."]], [Plain [Strong [Str "Image retrieval"], Str ": Store image embeddings and find visually similar images using cosine or L2 distance."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("table-definition", ["unnumbered", "unlisted"], []) [Str "Table Definition"], CodeBlock ("", ["sql"], []) "CREATE TABLE documents (
    id BIGSERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    embedding vector(1536)
);
", Header 3 ("insert-vectors", ["unnumbered", "unlisted"], []) [Str "Insert Vectors"], CodeBlock ("", ["sql"], []) "INSERT INTO documents (content, embedding)
VALUES ('Sample text', '[0.1, 0.2, 0.3, ...]');
", Header 3 ("nearest-neighbor-queries", ["unnumbered", "unlisted"], []) [Str "Nearest Neighbor Queries"], CodeBlock ("", ["sql"], []) "-- L2 distance (Euclidean)
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
", Header 3 ("create-indexes", ["unnumbered", "unlisted"], []) [Str "Create Indexes"], CodeBlock ("", ["sql"], []) "-- HNSW index with cosine distance
CREATE INDEX ON documents
USING hnsw (embedding vector_cosine_ops)
WITH (m = 16, ef_construction = 64);

-- IVFFlat index with L2 distance
CREATE INDEX ON documents
USING ivfflat (embedding vector_l2_ops)
WITH (lists = 100);
", Header 3 ("index-operator-classes", ["unnumbered", "unlisted"], []) [Str "Index Operator Classes"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.20833333333333334)), (AlignDefault, (ColWidth 0.1388888888888889)), (AlignDefault, (ColWidth 0.3194444444444444)), (AlignDefault, (ColWidth 0.3333333333333333))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Distance"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Operator"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "HNSW Operator Class"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "IVFFlat Operator Class"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "L2"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "<->"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "vector_l2_ops"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "vector_l2_ops"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Inner product"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "<#>"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "vector_ip_ops"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "vector_ip_ops"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Cosine"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "<=>"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "vector_cosine_ops"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "vector_cosine_ops"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "L1"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "<+>"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "vector_l1_ops"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "vector_l1_ops"]]]])] (TableFoot ("", [], []) []), Header 3 ("filtering-with-vector-search", ["unnumbered", "unlisted"], []) [Str "Filtering with Vector Search"], CodeBlock ("", ["sql"], []) "SELECT id, content
FROM documents
WHERE category = 'technical'
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'
LIMIT 10;
", Header 3 ("iterative-scans-v080", ["unnumbered", "unlisted"], []) [Str "Iterative Scans (v0.8.0+)"], CodeBlock ("", ["sql"], []) "-- Strict ordering: guarantees results are in exact distance order
SET hnsw.iterative_scan = strict_order;

SELECT id, content
FROM documents
WHERE category = 'technical'
ORDER BY embedding <=> '[0.1, 0.2, 0.3]'
LIMIT 10;

-- Relaxed ordering: faster but may return slightly out-of-order results
SET hnsw.iterative_scan = relaxed_order;
", Header 3 ("aggregate-functions", ["unnumbered", "unlisted"], []) [Str "Aggregate Functions"], CodeBlock ("", ["sql"], []) "-- Average of vectors
SELECT AVG(embedding) FROM documents;

-- Sum of vectors
SELECT SUM(embedding) FROM documents;
", Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("index-build-performance", ["unnumbered", "unlisted"], []) [Str "Index Build Performance"], CodeBlock ("", ["sql"], []) "-- Increase work memory for faster index builds
SET maintenance_work_mem = '2GB';

-- Parallelize index construction
SET max_parallel_maintenance_workers = 7;
", Header 3 ("hnsw-parameters", ["unnumbered", "unlisted"], []) [Str "HNSW Parameters"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.26744186046511625)), (AlignDefault, (ColWidth 0.10465116279069768)), (AlignDefault, (ColWidth 0.627906976744186))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Parameter"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Default"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Description"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "m"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "16"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Maximum number of connections per node in the graph"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "ef_construction"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "64"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Size of the dynamic candidate list during index build"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "hnsw.ef_search"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "40"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Size of the dynamic candidate list during query"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "hnsw.iterative_scan"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "off"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Enable iterative scan (", Code ("", [], []) "strict_order", Str " or ", Code ("", [], []) "relaxed_order", Str ")"]]]])] (TableFoot ("", [], []) []), Header 3 ("ivfflat-parameters", ["unnumbered", "unlisted"], []) [Str "IVFFlat Parameters"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.19)), (AlignDefault, (ColWidth 0.09)), (AlignDefault, (ColWidth 0.72))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Parameter"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Default"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Description"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "lists"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "-"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Number of inverted lists (clusters). Typical starting point: ", Code ("", [], []) "sqrt(N)"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "ivfflat.probes"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "1"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Number of lists to search at query time"]]]])] (TableFoot ("", [], []) []), Header 3 ("general-recommendations", ["unnumbered", "unlisted"], []) [Str "General Recommendations"], BulletList [[Plain [Str "For HNSW, increase ", Code ("", [], []) "hnsw.ef_search", Str " to improve recall at the cost of higher latency."]], [Plain [Str "For IVFFlat, increase ", Code ("", [], []) "ivfflat.probes", Str " to improve recall. A common starting point for ", Code ("", [], []) "lists", Str " is the square root of the total number of rows."]], [Plain [Str "Set ", Code ("", [], []) "maintenance_work_mem", Str " to at least 1 GB when building indexes on large tables."]], [Plain [Str "Use ", Code ("", [], []) "max_parallel_maintenance_workers", Str " to speed up index creation on multi-core systems."]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("python-with-psycopg", ["unnumbered", "unlisted"], []) [Str "Python with psycopg"], CodeBlock ("", ["python"], []) "import psycopg

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
embedding = [0.1, 0.2, 0.3]  # truncated for brevity
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
", Header 3 ("python-with-pgvector-python", ["unnumbered", "unlisted"], []) [Str "Python with pgvector-python"], CodeBlock ("", ["python"], []) "from pgvector.psycopg import register_vector
import psycopg
import numpy as np

conn = psycopg.connect(\"dbname=mydb\")
register_vector(conn)

embedding = np.array([0.1, 0.2, 0.3])
conn.execute(
    \"INSERT INTO documents (content, embedding) VALUES (%s, %s)\",
    (\"sample text\", embedding)
)
", Header 3 ("langchain-integration", ["unnumbered", "unlisted"], []) [Str "LangChain Integration"], CodeBlock ("", ["python"], []) "from langchain_community.vectorstores import PGVector

connection_string = \"postgresql://user:password@localhost:5432/mydb\"

vectorstore = PGVector.from_documents(
    documents=docs,
    embedding=embeddings_model,
    connection_string=connection_string,
    collection_name=\"my_collection\",
)

results = vectorstore.similarity_search(\"query text\", k=5)
", Header 3 ("sqlalchemy-with-pgvector", ["unnumbered", "unlisted"], []) [Str "SQLAlchemy with pgvector"], CodeBlock ("", ["python"], []) "from pgvector.sqlalchemy import Vector
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
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-rag-pipeline", ["unnumbered", "unlisted"], []) [Str "Basic RAG Pipeline"], CodeBlock ("", ["sql"], []) "-- Create table for document chunks
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
", Header 3 ("hybrid-search-with-full-text-and-vector", ["unnumbered", "unlisted"], []) [Str "Hybrid Search with Full-Text and Vector"], CodeBlock ("", ["sql"], []) "-- Table with both tsvector and vector columns
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
", Header 3 ("distance-threshold-query", ["unnumbered", "unlisted"], []) [Str "Distance Threshold Query"], CodeBlock ("", ["sql"], []) "-- Find all vectors within a cosine distance threshold
SELECT id, content, embedding <=> :query_embedding AS distance
FROM documents
WHERE embedding <=> :query_embedding < 0.3
ORDER BY embedding <=> :query_embedding;
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "vector(n) dimension limit"], Str ": Dense vectors are limited to 2,000 dimensions. Models producing higher-dimensional embeddings require dimensionality reduction or use of ", Code ("", [], []) "halfvec", Str " (up to 4,000 dimensions)."]], [Plain [Strong [Str "Approximate recall"], Str ": HNSW and IVFFlat indexes provide approximate results. Recall depends on index parameters and may not reach 100% without exact (sequential) scan."]], [Plain [Strong [Str "Index build time"], Str ": HNSW index construction can be slow on large datasets (millions of vectors). IVFFlat builds faster but requires the table to already contain data for effective clustering."]], [Plain [Strong [Str "Memory consumption"], Str ": HNSW indexes reside in memory and can be substantial for large datasets. Sizing depends on the number of vectors, dimensionality, and the ", Code ("", [], []) "m", Str " parameter."]], [Plain [Strong [Str "No built-in sharding"], Str ": pgvector relies on PostgreSQL's native partitioning and external sharding solutions (such as Citus) for horizontal scaling. It does not provide built-in distributed vector search."]], [Plain [Strong [Str "IVFFlat requires pre-populated data"], Str ": Building an IVFFlat index on an empty or very small table produces poor clusters. The table should contain a representative sample of data before index creation."]], [Plain [Strong [Str "Single-node scaling"], Str ": Performance is bounded by single-node PostgreSQL limits. For datasets exceeding hundreds of millions of vectors, purpose-built vector databases may offer better throughput."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "v0.8.0"], Str ": Added iterative scan modes (", Code ("", [], []) "strict_order", Str ", ", Code ("", [], []) "relaxed_order", Str ") for improved filtered vector search. Added ", Code ("", [], []) "sparsevec", Str " type."]], [Plain [Strong [Str "v0.7.0"], Str ": Added ", Code ("", [], []) "halfvec", Str " type for half-precision vectors. Added L1 distance operator (", Code ("", [], []) "<+>", Str ")."]], [Plain [Strong [Str "v0.6.0"], Str ": Added HNSW index support. Previously only IVFFlat was available."]], [Plain [Strong [Str "v0.5.0"], Str ": Added parallel index builds. Added ", Code ("", [], []) "bit", Str " type with Hamming and Jaccard distance operators."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " pgvector GitHub repository: https://github.com/pgvector/pgvector"]]]]