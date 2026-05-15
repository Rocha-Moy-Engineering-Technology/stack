# Qdrant

> Open-source AI-native vector database written in Rust for high-performance similarity search at scale

| Field | Value |
|-------|-------|
| Name | Qdrant |
| Group | Vector Databases |
| Type | SDK/Infra |
| Open Source | yes |
| GitHub | [qdrant/qdrant](https://github.com/qdrant/qdrant) |
| Stars | 31313 |
| Docs | [qdrant.tech/documentation](https://qdrant.tech/documentation/) |

## Overview

Qdrant (pronounced "quadrant") is an open-source vector database and semantic search engine written in Rust. It stores, indexes, and searches high-dimensional vector embeddings with associated metadata payloads, enabling similarity search that goes beyond keyword matching. Founded in 2021 and based in Berlin, Qdrant provides fast, scalable vector similarity search with convenient APIs [1].

The platform converts unstructured data (text, images, audio) into dense vector embeddings using embedding models, mapping them into high-dimensional space where semantically similar items cluster together. Qdrant combines dense vectors for contextual understanding with sparse vectors for precise lexical keyword matching through hybrid retrieval [1].

Qdrant offers four deployment options [1]:

- **Open Source (Self-Hosted)**: Full control over data and deployment with Docker, Kubernetes, or binary
- **Qdrant Cloud (Managed)**: Fully managed service with high availability and zero-downtime upgrades
- **Hybrid Cloud**: Managed control plane with data remaining in the user's infrastructure
- **Private Cloud**: Fully isolated deployment on user's own infrastructure

Official client libraries are available for Python, JavaScript/TypeScript, Rust, Go, Java, and .NET, with both REST (port 6333) and gRPC (port 6334) API interfaces [5].

## Core Concepts

### Collections

A **collection** is a named set of points (vectors with payloads) among which you search. All vectors within a collection must share the same dimensionality and distance metric, though named vectors allow multiple vector types per point with independent configurations. Collections support four distance metrics: Dot product, Cosine similarity, Euclidean distance, and Manhattan distance [2].

### Points

A **point** is the central entity consisting of three components: an identifier (64-bit unsigned integer or UUID), a vector (dense, sparse, or multi-vector), and an optional payload (metadata). Points are the fundamental storage unit combining vector embeddings with arbitrary JSON metadata [3].

### Vectors

Qdrant supports multiple vector types [3]:

- **Dense vectors**: Standard floating-point embeddings (Float32, Uint8) from neural network models
- **Sparse vectors**: High-dimensional vectors with mostly zero values, represented as index-value pairs for keyword-based search (BM25-style)
- **Multi-vectors**: Matrices from late-interaction models like ColBERT, where each document is represented as multiple vector chunks
- **Named vectors**: Multiple independently configured vectors per point, enabling storage of different embedding types (e.g., image and text) in a single collection

### Payloads

**Payloads** are arbitrary JSON metadata attached to points. Qdrant supports payload types including keyword (string), integer, float, bool, geo coordinates, datetime, text (full-text indexed), and UUID. Payload indexes extend the HNSW graph, enabling filtering criteria during the semantic search phase in a single-pass traversal rather than separate pre/post-filtering steps [1][4].

### Segments

Collections organize data into **segments**, each with independent vector storage, payload storage, indexes, and an ID mapper. Segments are either appendable (full CRUD) or non-appendable (read and delete only). Data integrity is maintained through a Write-Ahead Log (WAL) that orders operations sequentially before propagating to segments [8].

## Architecture

Qdrant uses a client-server architecture with distributed clustering capabilities:

```
┌─────────────────────────────────────────────────┐
│                Client SDKs                       │
│  Python, JS/TS, Rust, Go, Java, .NET            │
└──────────┬─────────────────┬────────────────────┘
           │ REST :6333      │ gRPC :6334
           v                 v
┌─────────────────────────────────────────────────┐
│              Qdrant Node(s)                      │
│  ┌─────────────────────────────────────────┐    │
│  │         Raft Consensus (Cluster)         │    │
│  │    (topology + collection structure)     │    │
│  └─────────────────────────────────────────┘    │
│  ┌──────────────┐  ┌──────────────────────┐    │
│  │  Collection   │  │  Collection          │    │
│  │  ┌─────────┐  │  │  ┌─────────┐        │    │
│  │  │ Shard 1 │  │  │  │ Shard 1 │        │    │
│  │  │┌───────┐│  │  │  │┌───────┐│        │    │
│  │  ││Segment││  │  │  ││Segment││        │    │
│  │  ││ HNSW  ││  │  │  ││ HNSW  ││        │    │
│  │  ││ WAL   ││  │  │  ││ WAL   ││        │    │
│  │  │└───────┘│  │  │  │└───────┘│        │    │
│  │  └─────────┘  │  │  └─────────┘        │    │
│  │  ┌─────────┐  │  │  ┌─────────┐        │    │
│  │  │ Shard 2 │  │  │  │ Shard 2 │        │    │
│  │  └─────────┘  │  │  └─────────┘        │    │
│  └──────────────┘  └──────────────────────┘    │
│                                                  │
│  ┌──────────────────────────────────────────┐   │
│  │           Storage Engine (Rust)           │   │
│  │  In-Memory / Memmap / On-Disk / RocksDB  │   │
│  └──────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

In distributed mode, Qdrant uses **Raft consensus** for cluster topology and collection structure operations. Point operations (insert, search) bypass consensus for low overhead. Collections split into **shards** distributed across nodes via consistent hashing. **Replication** creates shard copies across nodes, with configurable write consistency factors and read consistency levels (all, majority, quorum) [6].

### Vector Storage Options

- **In-Memory**: All vectors in RAM for maximum speed; disk used only for persistence
- **Memmap**: Memory-mapped files using page cache for near in-memory performance with lower RAM usage
- **On-Disk**: Full disk-based storage for datasets exceeding available memory [8]

## Key Features

- **HNSW Index**: Hierarchical Navigable Small World graph for fast approximate nearest neighbor search with configurable m, ef_construct, and ef parameters
- **Filterable HNSW**: Payload indexes extend HNSW graph edges, enabling single-pass filtered vector search without separate pre/post-filtering
- **Hybrid Search**: Combine dense and sparse vector queries with Reciprocal Rank Fusion (RRF) or Distribution-Based Score Fusion (DBSF) via the prefetch mechanism
- **Multi-Stage Search**: Nested prefetch pipelines for re-scoring — retrieve candidates with compact vectors, then re-rank with full-precision or ColBERT multi-vectors
- **Quantization**: Scalar (4x compression), Binary (up to 32x compression, 40x speedup), and Product quantization (up to 64x compression) with configurable rescoring and oversampling
- **Named Vectors**: Store multiple independently configured vector types per point (e.g., image and text embeddings)
- **Sparse Vectors**: Native sparse vector support with IDF modifier for keyword-based search
- **Rich Filtering**: Boolean clauses (must, should, must_not), range, geo (bounding box, radius, polygon), full-text match, datetime, nested object filters
- **Distributed Clustering**: Sharding with consistent hashing, replication, Raft consensus, and three shard transfer methods (stream_records, snapshot, wal_delta)
- **Collection Aliases**: Zero-downtime model upgrades by atomically switching collection pointers
- **ACORN Search**: Enhanced HNSW exploration for restrictive multi-filter queries via second-hop neighbor traversal
- **Grouping API**: Aggregate search results by payload field to avoid redundant items
- **Batch Operations**: Execute multiple operations (upsert, delete, update vectors, set payload) in a single request
- **Conditional Updates**: Optimistic locking with version-based filters to prevent concurrent overwrites
- **FastEmbed**: Built-in embedding generation library for client-side inference
- **Web UI Dashboard**: Built-in web interface for collection management and query exploration
- **MCP Server**: Model Context Protocol server for AI assistant integration

## Use Cases

- **Semantic Search**: Find documents, products, or media by meaning rather than exact keywords using dense vector similarity
- **Retrieval-Augmented Generation (RAG)**: Store document chunk embeddings and retrieve relevant context for LLM prompts with hybrid dense+sparse search
- **Recommendation Systems**: Find similar items based on user behavior embeddings with payload-based filtering for business rules
- **Image and Video Search**: Index visual embeddings for reverse image search and content-based retrieval
- **Anomaly Detection**: Identify outliers by measuring vector distances from normal behavior patterns
- **Multi-Tenant Applications**: Serve millions of users with payload-based partitioning or user-defined custom sharding for strict isolation
- **E-Commerce**: Product search combining visual similarity, text descriptions, and metadata filtering (price, category, availability)
- **Content Deduplication**: Detect near-duplicate documents or media using vector similarity thresholds

## API Reference

### Collection Operations

```python
from qdrant_client import QdrantClient, models

client = QdrantClient(url="http://localhost:6333")

# Create collection
client.create_collection(
    collection_name="my_collection",
    vectors_config=models.VectorParams(size=768, distance=models.Distance.COSINE),
)

# Multi-vector collection
client.create_collection(
    collection_name="multi_vec",
    vectors_config={
        "image": models.VectorParams(size=512, distance=models.Distance.DOT),
        "text": models.VectorParams(size=768, distance=models.Distance.COSINE),
    },
)

# Check existence
client.collection_exists("my_collection")

# Get collection info
client.get_collection("my_collection")
```

### Upserting Points

```python
client.upsert(
    collection_name="my_collection",
    wait=True,
    points=[
        models.PointStruct(
            id=1,
            vector=[0.05, 0.61, 0.76, 0.74],
            payload={"city": "Berlin", "category": "travel"},
        ),
        models.PointStruct(
            id=2,
            vector=[0.19, 0.81, 0.75, 0.11],
            payload={"city": "London", "category": "business"},
        ),
    ],
)
```

### Vector Search

```python
# Basic search
results = client.query_points(
    collection_name="my_collection",
    query=[0.2, 0.1, 0.9, 0.7],
    limit=5,
    with_payload=True,
)

# Filtered search
results = client.query_points(
    collection_name="my_collection",
    query=[0.2, 0.1, 0.9, 0.7],
    query_filter=models.Filter(
        must=[
            models.FieldCondition(
                key="city",
                match=models.MatchValue(value="London"),
            )
        ]
    ),
    limit=5,
)

# Search with parameters
results = client.query_points(
    collection_name="my_collection",
    query=[0.2, 0.1, 0.9, 0.7],
    search_params=models.SearchParams(hnsw_ef=128, exact=False),
    limit=10,
)
```

### Hybrid Search (Dense + Sparse)

```python
results = client.query_points(
    collection_name="my_collection",
    prefetch=[
        models.Prefetch(
            query=models.SparseVector(indices=[1, 42], values=[0.22, 0.8]),
            using="sparse",
            limit=20,
        ),
        models.Prefetch(
            query=[0.01, 0.45, 0.67],
            using="dense",
            limit=20,
        ),
    ],
    query=models.RrfQuery(rrf=models.Rrf(weights=[3.0, 1.0])),
)
```

### Payload Filtering

```python
# Scroll with complex filter
results = client.scroll(
    collection_name="my_collection",
    scroll_filter=models.Filter(
        must=[
            models.FieldCondition(key="city", match=models.MatchValue(value="Berlin")),
        ],
        must_not=[
            models.FieldCondition(key="category", match=models.MatchValue(value="spam")),
        ],
    ),
    limit=10,
    with_payload=True,
)

# Range filter
models.FieldCondition(key="price", range=models.Range(gte=100.0, lte=450.0))

# Geo filter
models.FieldCondition(
    key="location",
    geo_radius=models.GeoRadius(
        center=models.GeoPoint(lon=13.403683, lat=52.520711),
        radius=1000.0,
    ),
)
```

### Delete Operations

```python
# Delete by IDs
client.delete(
    collection_name="my_collection",
    points_selector=models.PointIdsList(points=[0, 3, 100]),
)

# Delete by filter
client.delete(
    collection_name="my_collection",
    points_selector=models.FilterSelector(
        filter=models.Filter(
            must=[models.FieldCondition(key="city", match=models.MatchValue(value="London"))]
        )
    ),
)
```

## Configuration

### Collection-Level Configuration

```python
client.create_collection(
    collection_name="optimized",
    vectors_config=models.VectorParams(
        size=768,
        distance=models.Distance.COSINE,
        on_disk=True,  # Memmap storage
    ),
    hnsw_config=models.HnswConfigDiff(m=16, ef_construct=100),
    optimizers_config=models.OptimizersConfigDiff(indexing_threshold=20000),
    quantization_config=models.ScalarQuantization(
        scalar=models.ScalarQuantizationConfig(
            type=models.ScalarType.INT8,
            quantile=0.99,
            always_ram=True,
        ),
    ),
)
```

### Payload Index Configuration

```python
# Keyword index
client.create_payload_index(
    collection_name="my_collection",
    field_name="category",
    field_schema=models.PayloadSchemaType.KEYWORD,
)

# Full-text index with tokenization
client.create_payload_index(
    collection_name="my_collection",
    field_name="description",
    field_schema=models.TextIndexParams(
        type="text",
        tokenizer=models.TokenizerType.WORD,
        min_token_len=2,
        max_token_len=15,
        lowercase=True,
    ),
)

# Tenant-optimized index
client.create_payload_index(
    collection_name="my_collection",
    field_name="tenant_id",
    field_schema=models.KeywordIndexParams(
        type="keyword",
        is_tenant=True,
    ),
)
```

### Server Configuration (YAML)

Key `config.yaml` parameters:

```yaml
storage:
  hnsw_index:
    m: 16
    ef_construct: 100
    full_scan_threshold: 10000
  on_disk_payload: false
  performance:
    max_search_threads: 0  # auto-detect

service:
  host: 0.0.0.0
  http_port: 6333
  grpc_port: 6334

cluster:
  enabled: false
  p2p:
    port: 6335
```

## Integration Patterns

### FastEmbed (Built-in Embeddings)

Qdrant maintains FastEmbed, a lightweight embedding generation library. Install with `pip install qdrant-client[fastembed]` for client-side inference without external API calls [1].

### LangChain Integration

Qdrant provides a native LangChain vector store for RAG pipelines, enabling document embedding storage and retrieval during LLM prompt construction.

### LlamaIndex Integration

LlamaIndex supports Qdrant as a vector store backend for document indexing and retrieval in RAG applications.

### MCP Server

Qdrant provides a Model Context Protocol server (`mcp-server-qdrant`) for connecting AI coding assistants and agents directly to Qdrant collections [1].

### Qdrant Edge

A lightweight deployment mode for edge computing environments with resource constraints [1].

## Examples

### RAG Pipeline with Hybrid Search

```python
from qdrant_client import QdrantClient, models

client = QdrantClient(url="http://localhost:6333")

# Create collection with dense + sparse vectors
client.create_collection(
    collection_name="rag_docs",
    vectors_config={
        "dense": models.VectorParams(size=768, distance=models.Distance.COSINE),
    },
    sparse_vectors_config={
        "sparse": models.SparseVectorParams(),
    },
)

# Insert document chunks
client.upsert(
    collection_name="rag_docs",
    points=[
        models.PointStruct(
            id=1,
            vector={
                "dense": [0.1, 0.2, ...],  # from embedding model
                "sparse": models.SparseVector(indices=[5, 10, 42], values=[0.5, 0.3, 0.8]),
            },
            payload={"text": "Document chunk content", "source": "docs/intro.md"},
        ),
    ],
)

# Hybrid search combining semantic + keyword
results = client.query_points(
    collection_name="rag_docs",
    prefetch=[
        models.Prefetch(query=[0.1, 0.2, ...], using="dense", limit=20),
        models.Prefetch(
            query=models.SparseVector(indices=[5, 42], values=[0.5, 0.8]),
            using="sparse",
            limit=20,
        ),
    ],
    query=models.RrfQuery(rrf=models.Rrf()),
    with_payload=True,
    limit=5,
)
```

### Multi-Tenant Collection

```python
# Create collection with tenant optimization
client.create_collection(
    collection_name="multi_tenant",
    vectors_config=models.VectorParams(size=768, distance=models.Distance.COSINE),
)

# Create tenant-optimized index
client.create_payload_index(
    collection_name="multi_tenant",
    field_name="tenant_id",
    field_schema=models.KeywordIndexParams(type="keyword", is_tenant=True),
)

# Insert tenant-specific data
client.upsert(
    collection_name="multi_tenant",
    points=[
        models.PointStruct(
            id=1,
            vector=[0.1, 0.2, ...],
            payload={"tenant_id": "company_a", "text": "Tenant A document"},
        ),
    ],
)

# Search scoped to tenant
results = client.query_points(
    collection_name="multi_tenant",
    query=[0.1, 0.2, ...],
    query_filter=models.Filter(
        must=[models.FieldCondition(key="tenant_id", match=models.MatchValue(value="company_a"))]
    ),
    limit=10,
)
```

### Quantized Collection for Large-Scale Deployment

```python
# Binary quantization for high-dimensional embeddings (32x compression)
client.create_collection(
    collection_name="large_scale",
    vectors_config=models.VectorParams(
        size=1536,
        distance=models.Distance.COSINE,
        on_disk=True,
    ),
    quantization_config=models.BinaryQuantization(
        binary=models.BinaryQuantizationConfig(
            always_ram=True,
        ),
    ),
)

# Search with rescoring for quality
results = client.query_points(
    collection_name="large_scale",
    query=[0.1, 0.2, ...],
    search_params=models.SearchParams(
        quantization=models.QuantizationSearchParams(
            rescore=True,
            oversampling=2.0,
        ),
    ),
    limit=10,
)
```

## Limitations

- **No ACID transactions**: Point operations bypass Raft consensus for performance; distributed atomicity is limited to single-point operations
- **Single distance metric per vector**: All dense vectors within a collection (or named vector) must use the same distance metric and dimensionality
- **Sparse vector constraints**: Sparse vectors support only dot-product distance and always use exact matching (no approximate indexing)
- **Two-node cluster limitations**: Collection create/edit/delete operations fail when one node is offline since recovery requires >50% of nodes healthy [6]
- **No built-in embedding**: Qdrant stores and searches vectors but does not generate embeddings (FastEmbed is a separate client-side library); server-side inference is limited to Qdrant Cloud
- **Product quantization performance**: PQ is slower than scalar quantization (non-SIMD-friendly) with significant accuracy loss (~0.7 accuracy) [7]
- **Memory requirements**: In-memory HNSW indexes require substantial RAM for large datasets; memmap or on-disk storage must be explicitly configured
- **Default no authentication**: Qdrant starts with no encryption or authentication by default; security must be explicitly configured [9]
- **Eventual consistency default**: Distributed deployments prioritize availability and throughput; strong consistency requires explicit configuration of write ordering and read consistency levels [6]

## Changelog

- **v1.17.0**: RRF weighted fusion, update modes (insert_only, update_only), disable HNSW edges per field
- **v1.16.0**: ACORN search algorithm, conditional updates with filters, collection metadata, ASCII folding in text indexes, full-text any matching
- **v1.15.0**: 1.5-bit and 2-bit binary quantization, asymmetric quantization with query encoding
- **v1.13.0**: Resharding (Cloud), has_vector filter condition
- **v1.11.0**: Distribution-Based Score Fusion (DBSF), on-disk payload indexes, tenant/principal index types, UUID matching, grouping in hybrid queries
- **v1.10.0**: Sparse vector IDF modifier
- **v1.9.0**: Uint8 vector datatype support
- **v1.8.0**: WAL delta shard transfer, datetime range filters, order-by payload scrolling, parameterized integer indexes
- **v1.7.0**: Sparse vector support, user-defined custom sharding, snapshot shard transfer
- **v1.5.0**: Binary quantization, batch update operations

## Citations

- [1] Overview - https://qdrant.tech/documentation/overview/
- [2] Collections - https://qdrant.tech/documentation/concepts/collections/
- [3] Points - https://qdrant.tech/documentation/concepts/points/
- [4] Filtering - https://qdrant.tech/documentation/concepts/filtering/
- [5] API & SDKs - https://qdrant.tech/documentation/interfaces/
- [6] Distributed Deployment - https://qdrant.tech/documentation/guides/distributed_deployment/
- [7] Quantization - https://qdrant.tech/documentation/guides/quantization/
- [8] Storage - https://qdrant.tech/documentation/concepts/storage/
- [9] Quickstart - https://qdrant.tech/documentation/quickstart/
- [10] Search - https://qdrant.tech/documentation/concepts/search/
- [11] Hybrid Queries - https://qdrant.tech/documentation/concepts/hybrid-queries/
- [12] Indexing - https://qdrant.tech/documentation/concepts/indexing/

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

Qdrant, Rust, vector database, collections, points, payload, payload index, filterable HNSW, ACORN, multi-vector, ColBERT, sparse vectors, dense vectors, named vectors, hybrid search, RRF, DBSF, prefetch, multi-stage search, scalar quantization, binary quantization, product quantization, FastEmbed, MCP server, Raft consensus, sharding, replication, REST 6333, gRPC 6334, hybrid cloud, private cloud

### Verb-Noun Tasks

- Create a collection with `VectorParams` (size, distance)
- Upsert points with vectors and JSON payloads
- Query with `query_points` and filters
- Build a payload index for single-pass filtered HNSW search
- Run hybrid search via `prefetch` with RRF or DBSF fusion
- Re-rank with ColBERT multi-vectors in a nested prefetch
- Apply binary quantization for 32x compression
- Configure rescoring and oversampling for quantized search
- Use ACORN for restrictive multi-filter queries
- Use FastEmbed for client-side inference
- Run `mcp-server-qdrant` for AI assistant integration
- Shard collections via consistent hashing across a Raft cluster

### User Intent Phrases

- How do I do filtered vector search without the post-filter penalty?
- How do I run hybrid dense + sparse search with RRF fusion?
- How do I add a ColBERT multi-vector reranking stage?
- How do I compress vectors 32x with binary quantization?
- How do I self-host a high-performance vector database in Rust?
- How do I expose my collection to an AI assistant via MCP?
- How do I isolate tenants with payload indexes?
- How do I configure HNSW parameters (m, ef_construct, ef)?
- How do I run nested prefetch pipelines for multi-stage retrieval?

### Problem Statements

- Filtered HNSW search degrades when filters are restrictive
- Combining dense and sparse search requires manual fusion logic
- Multi-stage re-scoring with full-precision vectors is hard to express
- Self-hosted vector DBs in Java/Go can be slower than expected
- Multi-tenant workloads need data-level isolation without per-tenant collections
- Connecting AI assistants to vector data is glue work
- Default deployments lack auth and encryption

### When to Pick This

- Pick this when restrictive metadata filtering with single-pass HNSW (payload-indexed) matters at scale
- Pick this over Pinecone when self-hosting in Rust with high performance and full control is required
- Pick this over Weaviate when nested prefetch pipelines and ColBERT multi-vector re-scoring fit your retrieval shape
- Pick this over Milvus when Rust performance + filterable HNSW + ACORN matter more than billion-scale GPU-accelerated indexes
- Pick this over pgvector when you have no existing PostgreSQL footprint and want a purpose-built engine
- Pick this when MCP-server integration with AI coding assistants is on your roadmap

### Related Terms and Aliases

- Qdrant Cloud / Hybrid Cloud / Private Cloud / Edge
- qdrant-client (Python)
- FastEmbed
- Filterable HNSW
- ACORN algorithm
- RRF / DBSF fusion
- Multi-vector (ColBERT)
- Payload (metadata)
- Prefetch
- mcp-server-qdrant
- WAL (write-ahead log)
- Raft consensus
