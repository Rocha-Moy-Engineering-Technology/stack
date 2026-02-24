# Pinecone

> The leading vector database for building accurate and performant AI applications at scale in production. Fully managed infrastructure with integrated embedding, semantic/lexical/hybrid search, metadata filtering, and reranking.

| Field | Value |
|-------|-------|
| Name | Pinecone |
| Group | RAG & Knowledge Retrieval |
| Type | API/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Docs | https://docs.pinecone.io/ |

## Overview

Pinecone is a fully managed vector database purpose-built for AI applications that require fast, accurate similarity search at production scale. Unlike self-hosted vector databases that demand infrastructure expertise, Pinecone abstracts away the operational complexity of indexing, sharding, replication, and scaling, exposing a clean API for storing, querying, and managing high-dimensional vector embeddings.

The platform supports three search paradigms: dense vector search for semantic similarity, sparse vector search for lexical/keyword matching, and hybrid search that combines both approaches. With integrated embedding capabilities, Pinecone can accept raw text and automatically convert it to vectors, eliminating the need for external embedding pipelines.

Pinecone serves as a retrieval backbone for Retrieval-Augmented Generation (RAG) systems, recommendation engines, anomaly detection, and any application where finding semantically similar items in large corpora is critical. Its namespace mechanism provides built-in multitenancy and data partitioning without requiring separate indexes.

## Core Concepts

### Indexes

An index is the primary organizational unit in Pinecone. Each index stores vectors and their associated metadata, and it is configured at creation time with a specific dimensionality and distance metric. Pinecone offers two index types:

- **Dense indexes** store floating-point vector embeddings for semantic similarity search. Vectors are compared using cosine similarity, Euclidean distance, or dot product.
- **Sparse indexes** store sparse vector representations for lexical and keyword-based search, analogous to traditional information retrieval techniques like BM25.

### Vectors

A vector is an array of floating-point numbers representing an embedded piece of data (text, image, audio, etc.). Each vector is identified by a unique ID and can carry arbitrary key-value metadata. Vectors are upserted (inserted or updated) into an index and retrieved through query operations.

### Namespaces

Namespaces partition vectors within a single index. Every query and upsert operation targets a specific namespace (or the default namespace if none is specified). Namespaces enable multitenant architectures where each tenant's data is logically isolated without the overhead of maintaining separate indexes. Queries never cross namespace boundaries.

### Metadata

Each vector can carry a metadata dictionary with string, numeric, boolean, or list-of-strings values. Metadata filters can be applied during queries to narrow results beyond vector similarity, enabling filtered semantic search (e.g., "find similar documents published after 2024 in the 'engineering' category").

### Integrated Embedding

Pinecone's integrated embedding feature accepts raw text input and automatically generates vector embeddings using a built-in model. This eliminates the need for a separate embedding service, reducing latency and architectural complexity for text-based use cases.

### Reranking

After an initial retrieval pass, Pinecone can rerank results using a cross-encoder or similar model to improve precision. Reranking is particularly useful in RAG pipelines where the top-k retrieved documents must be highly relevant before being passed to a Large Language Model (LLM).

## Installation

### Python SDK

```bash
pip install pinecone
```

### Node.js SDK

```bash
npm install @pinecone-database/pinecone
```

### CLI

Pinecone provides a Command-Line Interface (CLI) for index management, data operations, and account administration:

```bash
pip install pinecone-cli
pinecone login
```

### Authentication

All API access requires an API key, obtained from the Pinecone console. The key is passed via the `Api-Key` header in REST calls or through SDK client initialization:

```python
from pinecone import Pinecone

pc = Pinecone(api_key="YOUR_API_KEY")
```

## Architecture

### Fully Managed Infrastructure

Pinecone handles all infrastructure concerns: provisioning, scaling, replication, backups, and failover. Users interact exclusively through APIs and SDKs without managing servers, containers, or storage volumes.

### Storage and Compute Separation

Indexes in Pinecone separate storage from compute. Data is durably persisted independently of query-serving nodes, enabling scaling of read throughput without re-ingesting data.

### Pod-Based and Serverless Deployments

Pinecone offers two deployment models:

- **Serverless** indexes scale automatically based on usage, with no provisioning required. Billing is based on storage and query volume.
- **Pod-based** indexes provide dedicated compute resources with predictable performance characteristics, suited for workloads requiring consistent low-latency guarantees.

### Namespace Isolation

Namespaces provide logical isolation within a single index. Each namespace maintains its own vector set, and queries are scoped to a single namespace. This architecture supports multitenancy at the data layer without the cost and complexity of per-tenant indexes.

## Key Features

- **Semantic search**: Query dense indexes with vector embeddings to find semantically similar items regardless of keyword overlap.
- **Lexical search**: Query sparse indexes for keyword-based retrieval, suitable for exact-match and traditional information retrieval workloads.
- **Hybrid search**: Combine dense and sparse search in a single query to balance semantic understanding with keyword precision.
- **Integrated embedding**: Send raw text directly to Pinecone and let the platform handle embedding generation, removing the need for an external embedding service.
- **Metadata filtering**: Apply filters on metadata fields during queries to scope results by category, date range, source, or any custom attribute.
- **Reranking**: Reorder initial retrieval results using a cross-encoder model to improve precision before downstream consumption.
- **Upsert and batch import**: Insert or update vectors individually or in bulk. Batch import supports large-scale data ingestion from cloud storage.
- **Namespace organization**: Partition data within an index for multitenancy, access control, or logical separation of datasets.
- **Real-time updates**: Upserted vectors become queryable with low latency, enabling near-real-time retrieval over changing data.
- **Managed scaling**: Serverless indexes scale automatically; pod-based indexes can be resized without downtime.

## Use Cases

### Retrieval-Augmented Generation

Pinecone serves as the retrieval layer in RAG systems. Documents are chunked, embedded, and stored in an index. At query time, the user's question is embedded and used to retrieve the most relevant chunks, which are then passed as context to an LLM for grounded answer generation. Metadata filtering narrows retrieval to specific document sets, time ranges, or access levels.

### Semantic Search

Applications that need to find items by meaning rather than exact keywords use Pinecone's dense indexes. Examples include searching knowledge bases, FAQs, support tickets, and product catalogs where user queries are natural-language and rarely match document text verbatim.

### Question Answering Over Proprietary Data

Pinecone's assistant quickstart demonstrates building a Q&A system over proprietary documents. Documents are ingested, and users ask natural-language questions that retrieve relevant passages for LLM-powered answers grounded in the organization's own data.

### Recommendation Systems

By embedding users and items into the same vector space, Pinecone enables real-time similarity-based recommendations. Querying with a user's embedding returns the most similar items, and metadata filters can enforce business rules (e.g., only recommend in-stock products).

### Anomaly Detection

Vectors representing normal behavior are indexed, and incoming data points are queried against the index. Points with low similarity to any stored vector are flagged as anomalies. This pattern applies to fraud detection, network security, and manufacturing quality control.

## API Reference

### Index Management

```python
from pinecone import Pinecone, ServerlessSpec

pc = Pinecone(api_key="YOUR_API_KEY")

# Create a serverless index
pc.create_index(
    name="my-index",
    dimension=1536,
    metric="cosine",
    spec=ServerlessSpec(cloud="aws", region="us-east-1")
)

# List indexes
indexes = pc.list_indexes()

# Describe an index
description = pc.describe_index("my-index")

# Delete an index
pc.delete_index("my-index")
```

### Vector Operations

```python
index = pc.Index("my-index")

# Upsert vectors
index.upsert(
    vectors=[
        {
            "id": "doc-1",
            "values": [0.1, 0.2, ...],  # 1536-dimensional vector
            "metadata": {"source": "wiki", "category": "science"}
        },
        {
            "id": "doc-2",
            "values": [0.3, 0.4, ...],
            "metadata": {"source": "arxiv", "category": "engineering"}
        }
    ],
    namespace="tenant-a"
)

# Query with metadata filter
results = index.query(
    vector=[0.15, 0.25, ...],
    top_k=10,
    namespace="tenant-a",
    filter={"category": {"$eq": "science"}},
    include_metadata=True
)

# Fetch vectors by ID
fetched = index.fetch(ids=["doc-1", "doc-2"], namespace="tenant-a")

# Delete vectors
index.delete(ids=["doc-1"], namespace="tenant-a")
```

### Integrated Embedding

```python
# Upsert with raw text (integrated embedding)
index.upsert_records(
    namespace="tenant-a",
    records=[
        {"_id": "doc-1", "text": "Pinecone is a vector database.", "category": "tech"},
        {"_id": "doc-2", "text": "Vectors enable semantic search.", "category": "tech"}
    ]
)

# Query with raw text
results = index.search(
    namespace="tenant-a",
    query={"inputs": {"text": "How does semantic search work?"}, "top_k": 5}
)
```

## Configuration

### Index Configuration

| Parameter | Description | Options |
|-----------|-------------|---------|
| `name` | Unique index identifier | Alphanumeric string |
| `dimension` | Vector dimensionality (must match embedding model output) | Integer (e.g., 768, 1024, 1536) |
| `metric` | Distance metric for similarity | `cosine`, `euclidean`, `dotproduct` |
| `spec` | Deployment specification | Serverless or pod-based |

### Serverless Spec

| Parameter | Description | Options |
|-----------|-------------|---------|
| `cloud` | Cloud provider | `aws`, `gcp`, `azure` |
| `region` | Deployment region | Provider-specific region strings |

### Query Parameters

| Parameter | Description | Default |
|-----------|-------------|---------|
| `top_k` | Number of results to return | 10 |
| `namespace` | Target namespace | `""` |
| `filter` | Metadata filter expression | None |
| `include_metadata` | Return metadata with results | False |
| `include_values` | Return vector values with results | False |

## Integration Patterns

### LangChain

```python
from langchain_pinecone import PineconeVectorStore
from langchain_openai import OpenAIEmbeddings

vectorstore = PineconeVectorStore(
    index_name="my-index",
    embedding=OpenAIEmbeddings(),
    namespace="documents"
)

retriever = vectorstore.as_retriever(search_kwargs={"k": 5})
```

### LlamaIndex

```python
from llama_index.vector_stores.pinecone import PineconeVectorStore

vector_store = PineconeVectorStore(
    pinecone_index=index,
    namespace="documents"
)
```

### Direct REST API

```bash
curl -X POST "https://my-index-abc1234.svc.us-east-1.pinecone.io/query" \
  -H "Api-Key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "vector": [0.1, 0.2, ...],
    "topK": 10,
    "namespace": "tenant-a",
    "filter": {"category": {"$eq": "science"}},
    "includeMetadata": true
  }'
```

## Examples

### RAG Pipeline with Integrated Embedding

```python
from pinecone import Pinecone

pc = Pinecone(api_key="YOUR_API_KEY")
index = pc.Index("knowledge-base")

# Ingest documents
documents = [
    {"_id": "chunk-1", "text": "The mitochondria is the powerhouse of the cell.", "source": "biology-101"},
    {"_id": "chunk-2", "text": "ATP is produced through oxidative phosphorylation.", "source": "biology-101"},
    {"_id": "chunk-3", "text": "Photosynthesis converts light energy to chemical energy.", "source": "biology-101"},
]

index.upsert_records(namespace="biology", records=documents)

# Query
results = index.search(
    namespace="biology",
    query={"inputs": {"text": "How do cells produce energy?"}, "top_k": 3}
)

# Pass retrieved context to LLM
context = "\n".join([match["text"] for match in results["matches"]])
```

### Multitenant Search

```python
# Tenant A ingests their documents
index.upsert(
    vectors=[{"id": "a-1", "values": embedding_a1, "metadata": {"doc_type": "contract"}}],
    namespace="tenant-a"
)

# Tenant B ingests their documents
index.upsert(
    vectors=[{"id": "b-1", "values": embedding_b1, "metadata": {"doc_type": "invoice"}}],
    namespace="tenant-b"
)

# Queries are scoped to the tenant's namespace
results_a = index.query(vector=query_vec, top_k=5, namespace="tenant-a")
results_b = index.query(vector=query_vec, top_k=5, namespace="tenant-b")
# Tenant A never sees Tenant B's data and vice versa
```

### Hybrid Search

```python
# Query combining dense and sparse vectors
results = index.query(
    vector=dense_embedding,          # semantic component
    sparse_vector=sparse_embedding,  # lexical component
    top_k=10,
    namespace="documents"
)
```

## Limitations

- **Proprietary and closed-source**: No self-hosting option. All data is stored on Pinecone's managed infrastructure, which may not meet certain data sovereignty or air-gapped deployment requirements.
- **Vendor lock-in**: The API and data format are Pinecone-specific. Migrating to another vector database requires re-ingesting data and rewriting query logic.
- **Cost at scale**: Pricing is usage-based. Large-scale deployments with high query volumes and substantial storage can become expensive compared to self-hosted alternatives.
- **Index immutability**: Certain index properties (dimension, metric) cannot be changed after creation. Changing these requires creating a new index and re-ingesting all data.
- **Metadata filter constraints**: Metadata values are limited to specific types (string, number, boolean, list of strings). Complex nested objects are not supported as metadata values.
- **Namespace limitations**: Namespaces cannot be listed or enumerated via the API. Applications must track namespaces externally.
- **No server-side joins or aggregations**: Pinecone is a retrieval engine, not a general-purpose database. Complex queries involving joins, aggregations, or transactions are not supported.

## Changelog

- **Integrated embedding**: Pinecone added the ability to accept raw text and automatically generate embeddings, simplifying the ingestion pipeline for text-based use cases.
- **Serverless indexes**: Introduction of serverless deployment model with automatic scaling and usage-based billing alongside the original pod-based architecture.
- **Sparse indexes**: Support for sparse vector search enabling lexical/keyword retrieval alongside dense semantic search.
- **Reranking**: Built-in reranking capability to improve precision of retrieval results before downstream LLM consumption.
- **Assistant quickstart**: Addition of a guided quickstart for building Q&A systems over proprietary data.

## Citations

- [1] [Pinecone Documentation](https://docs.pinecone.io/)
