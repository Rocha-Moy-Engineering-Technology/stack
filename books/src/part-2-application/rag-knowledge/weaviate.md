# Weaviate

> Open-source vector database for storing data objects and vector embeddings with semantic search, hybrid search, and RAG capabilities

| Field | Value |
|-------|-------|
| Group | RAG & Knowledge Retrieval |
| Type | SDK/Infra |
| Open Source | Yes |
| GitHub | [https://github.com/weaviate/weaviate](https://github.com/weaviate/weaviate) |
| Stars | 15600 |
| Documentation | [Official Docs](https://docs.weaviate.io/weaviate/) |

## Overview

Weaviate is an open-source vector database designed to store and index both data objects and their vector embeddings. Written in Go, it enables semantic search by comparing meaning in vectors rather than relying solely on keyword matching, and supports hybrid search that combines both approaches. Weaviate serves as a backend for Retrieval Augmented Generation (RAG) workflows where vector search retrieves context that enhances the output of generative models, and it supports agent-driven workflows through its flexible API and AI model integration. [1]

The broader Weaviate ecosystem includes five components: the Weaviate Database (core open-source engine), Weaviate Cloud (fully managed deployment), Weaviate Agents (pre-built agentic services for query, transformation, and personalization tasks), Weaviate Embeddings (managed inference for vector generation), and Model Providers (third-party integrations with OpenAI, Cohere, Anthropic, Google, and others). [1]

## Core Concepts

### Collections and Objects

Each data object in Weaviate belongs to a collection and has one or more properties. Objects are stored as JSON documents within class-based collections sharing a common schema. Properties define object attributes with data types including strings, numbers, dates, and booleans. Every object receives a UUID guaranteeing uniqueness across collections, with support for deterministic UUIDs or automatic generation. [1]

### Vector Embeddings

Vectors are numerical arrays derived from machine learning models that represent the semantic meaning of data objects. Collections can have multiple named vectors with independent configurations, indexes, and compression algorithms. Weaviate can auto-generate vectors via integrated vectorizer modules or accept pre-computed vectors at import time. Text properties are processed in alphabetical order before vectorization. [1]

### Cross-References

Directional links that represent relationships between objects across collections. Cross-reference queries may impact performance at scale, so alternative schema designs should be considered for relationship-heavy workloads. [1]

### Multi-Tenancy

Data isolation mechanism that creates dedicated shards per tenant. Tenants have activity states: ACTIVE (accessible in memory), INACTIVE (local storage only), OFFLOADED (moved to cloud storage), OFFLOADING (transition), or ONLOADING (transition). This architecture supports approximately 50,000 or more active shards per node. [1]

### Schema

A formal blueprint defining collections, properties, cross-references, and vectorizer settings. Weaviate auto-generates schemas from incoming data if not explicitly defined, but explicit schemas are recommended for production use. [1]

## Installation and Setup

### Python

```bash
pip install -U weaviate-client
```

Connect to Weaviate Cloud:

```python
import weaviate
import os

weaviate_url = os.environ["WEAVIATE_URL"]
weaviate_api_key = os.environ["WEAVIATE_API_KEY"]

with weaviate.connect_to_weaviate_cloud(
    cluster_url=weaviate_url,
    auth_credentials=weaviate_api_key,
) as client:
    print(client.is_ready())
```
[1]

### TypeScript/JavaScript

```bash
npm install weaviate-client
```

```typescript
import weaviate, { WeaviateClient, ApiKey } from 'weaviate-client';

const client: WeaviateClient = await weaviate.connectToWeaviateCloud(
  process.env.WEAVIATE_URL!,
  { authCredentials: new ApiKey(process.env.WEAVIATE_API_KEY!) }
);
```
[1]

### Docker

Deploy locally with Docker Compose for development and testing:

```yaml
version: '3.4'
services:
  weaviate:
    image: cr.weaviate.io/semitechnologies/weaviate:latest
    ports:
      - "8080:8080"
      - "50051:50051"
    environment:
      QUERY_DEFAULTS_LIMIT: 25
      AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED: 'true'
      PERSISTENCE_DATA_PATH: '/var/lib/weaviate'
      ENABLE_MODULES: ''
      CLUSTER_HOSTNAME: 'node1'
```

Connect to local instance:

```python
import weaviate

with weaviate.connect_to_local() as client:
    print(client.is_ready())
```
[1]

### Kubernetes

Deploy using Helm charts with `values.yaml` configuration. Supports scaling from development through production with optional zero-downtime updates and local inference containers. [1]

### Embedded Weaviate

Launch directly from Python or JavaScript/TypeScript for quick evaluation without a separate server process. Experimental feature primarily intended for prototyping. [1]

## Architecture

### Storage Engine

Weaviate persists data through the interaction of three storage components within each shard: an inverted index (for keyword filtering and BM25 search), a vector index (for similarity search), and an object store (for the actual data objects). Each collection is divided into shards, and each shard manages its own set of these three components. [1]

### Vector Index Types

Three vector index algorithms are available:

- **HNSW (Hierarchical Navigable Small World)** -- The default index type. Uses multi-layered graphs with hierarchical layers containing exponentially fewer objects at each level, delivering logarithmic query complexity for large-scale deployments. Configurable via `ef`, `efConstruction`, and `maxConnections` parameters
- **Flat Index** -- A simple, lightweight index that is fast to build with a very small memory footprint. Linear time complexity makes it suitable only for small collections and multi-tenant scenarios
- **Dynamic Index** -- Experimental feature (v1.25+) that automatically switches from flat to HNSW when objects exceed a configurable threshold (default: 10,000), optimizing memory usage for growing collections [1]

### Vector Quantization

Compression options reduce memory requirements since index size in memory is directly proportional to vector count:

- **Product Quantization (PQ)** -- Segments vectors and quantizes each segment independently
- **Binary Quantization (BQ)** -- Converts floating-point values to binary representations
- **Scalar Quantization (SQ)** -- Reduces precision of individual vector components [1]

### Distance Metrics

All standard distance metrics are supported for vector similarity calculation: cosine, dot product, L2 (Euclidean), and Manhattan distance. [1]

### Horizontal Scaling

Weaviate supports two complementary scaling approaches:

- **Sharding** -- Partitions data across nodes for throughput scaling
- **Replication** -- Duplicates data across nodes for fault tolerance and availability

These can be combined: a collection can be both sharded and replicated across multiple nodes. Replication uses a leaderless design with no primary/secondary node distinction. [1]

### Replication Consistency

Three tunable consistency levels for read and write operations:

- **ONE** -- Minimal consistency; enables linear throughput scaling
- **QUORUM** -- Requires acknowledgment from n/2+1 nodes
- **ALL** -- Most strict; synchronous, all replicas must acknowledge [1]

### API Layer

Three API interfaces serve different purposes:

- **REST API** -- Collection management, CRUD operations, node status, backups, and cluster health monitoring
- **GraphQL API** -- Data querying with semantic search (`nearText`, `nearVector`), keyword search (`bm25`), hybrid search, filtering, and aggregations
- **gRPC API** -- High-performance alternative using Protocol Buffers for faster serialization and lower latency, focused on search operations and batch imports [1]

Modern client libraries (Python v4+, TypeScript v3+) automatically use the gRPC interface for search operations when available, handling protocol negotiation transparently. [1]

## Key Features and Functionality

### Semantic Vector Search

Find objects based on vector embedding similarity using `nearText` (text query vectorized automatically), `nearVector` (pre-computed vector), `nearObject` (vector of an existing object), and `nearImage` (image query):

```python
movies = client.collections.use("Movie")
response = movies.query.near_text(query="sci-fi adventure", limit=5)

for obj in response.objects:
    print(obj.properties)
```
[1]

### Keyword Search (BM25F)

Execute ranked keyword searches using the BM25F algorithm with property weighting:

```python
response = movies.query.bm25(query="matrix", limit=5)
```

Supports `or` operator (at least N tokens must match) and `and` operator (all tokens required). Properties can be boosted using syntax like `"title^2"` to increase their contribution to keyword scores. [1]

### Hybrid Search

Combines vector search and BM25F keyword search by fusing result sets. The `alpha` parameter controls the balance:

- `alpha=1.0` -- Pure vector search (semantic only)
- `alpha=0.0` -- Pure keyword search (lexical only)
- `alpha=0.5` -- Equal weighting (default)

```python
response = movies.query.hybrid(query="sci-fi adventure", alpha=0.75, limit=5)
```

Two fusion algorithms are available: Relative Score Fusion (default in v1.24+, uses actual similarity scores) and Ranked Fusion (uses ranking positions). [1]

### Retrieval Augmented Generation (RAG)

Integrate search results directly with generative models to augment LLM prompts with retrieved context:

```python
from weaviate.classes.generate import GenerativeConfig

response = movies.generate.near_text(
    query="sci-fi",
    limit=2,
    grouped_task="Summarize these movies in one paragraph.",
    generative_provider=GenerativeConfig.anthropic(model="claude-3-5-haiku-latest"),
)
print(response.generative.text)
```
[1]

### Weaviate Agents

Pre-built agentic services available to Weaviate Cloud users:

- **Query Agent** -- Responds to natural language questions by searching stored data and delivering answers. Translates plain English to optimized Weaviate queries automatically
- **Transformation Agent** (technical preview) -- Modifies and enhances datasets according to specified instructions
- **Personalization Agent** (technical preview) -- Tailors outputs to specific personas with the ability to learn and refine preferences over time [1]

### Reranking

Refine and re-order search results using integrated reranker modules from providers including Cohere, Jina AI, NVIDIA, and Voyage AI. [1]

### Filtering

Combine vector search with `where` clause filters. Weaviate merges HNSW with inverted indexes for high-recall, high-speed filtered queries, avoiding the performance degradation common in post-filtering approaches. [1]

### Multi-Target Vectors

Search across multiple named vector fields within a single collection, enabling multimodal search scenarios where objects have separate vectors for text, images, or other modalities. [1]

### Time-to-Live (TTL)

Added in v1.35.0 as a technical preview. Automatically expire and delete objects based on creation time, last update, or DATE property values. [1]

## Use Cases

### Semantic Search Applications

Build search systems that understand query intent and return relevant results even when query terms do not exactly match stored data. Combine with hybrid search for applications requiring both semantic understanding and exact term matching.

### RAG Pipelines

Use Weaviate as the vector store backend for RAG workflows. Vector search retrieves relevant context documents, which are then passed to generative models (OpenAI, Anthropic, Google, Cohere) to produce grounded, context-aware responses.

### Recommendation Systems

Leverage vector similarity to find items similar to user preferences or past interactions. Multi-tenancy isolates per-user data, and the Personalization Agent provides pre-built personalization capabilities.

### Knowledge Management

Store and index enterprise documents, articles, and knowledge base entries. Cross-references model relationships between entities. Filtering narrows results by metadata attributes.

### Multimodal Search

Support image, text, and combined search queries using multimodal vectorizer integrations (CLIP, ImageBind, Google multimodal, NVIDIA multimodal) that encode different data types into a shared vector space.

### Agent-Driven Applications

Power intelligent agents that leverage semantic search insights through the flexible API. The Query Agent enables natural language access to stored data without writing explicit queries.

## API Reference Summary

### REST Endpoints

- `GET /v1/schema` -- Retrieve all collection schemas
- `POST /v1/schema` -- Create a new collection
- `GET /v1/objects` -- List objects with optional filters
- `POST /v1/objects` -- Create a single object
- `POST /v1/batch/objects` -- Batch import objects
- `GET /v1/nodes` -- Node status and health
- `POST /v1/backups` -- Create backup
- `GET /v1/.well-known/ready` -- Readiness check
- `GET /v1/.well-known/live` -- Liveness check

### GraphQL Queries

- `Get` -- Retrieve objects with vector, keyword, or hybrid search
- `Aggregate` -- Perform aggregation operations (count, sum, mean)
- `Explore` -- Search across all collections

### Search Operators (GraphQL)

- `nearText` -- Semantic search with text query
- `nearVector` -- Search with pre-computed vector
- `nearObject` -- Search using existing object vector
- `nearImage` -- Search with image input
- `bm25` -- BM25F keyword search
- `hybrid` -- Combined vector and keyword search

### Client Libraries

- **Python** (`weaviate-client`) -- v4+ with gRPC support
- **TypeScript/JavaScript** (`weaviate-client`) -- v3+ with gRPC support
- **Go** (`weaviate-go-client`) -- REST and GraphQL
- **Java** (`io.weaviate:client`) -- REST and GraphQL [1]

## Configuration and Customization

### Core Settings

- **`PERSISTENCE_DATA_PATH`** -- Data storage location on disk
- **`QUERY_DEFAULTS_LIMIT`** -- Default number of results returned (default: 10 in v1.24+)
- **`QUERY_MAXIMUM_RESULTS`** -- Upper limit on retrievable objects per query
- **`GOMEMLIMIT`** -- Memory ceiling for Go runtime (recommended 80-90% of total)
- **`GOMAXPROCS`** -- Maximum concurrent OS threads
- **`DEFAULT_VECTORIZER_MODULE`** -- Default vectorizer for new collections
- **`DEFAULT_QUANTIZATION`** -- Default quantization technique (pq, bq, sq, none) [1]

### Authentication

- **API Key** -- `AUTHENTICATION_APIKEY_ENABLED` with `AUTHENTICATION_APIKEY_ALLOWED_KEYS`
- **OIDC** -- `AUTHENTICATION_OIDC_ENABLED` with issuer, client ID, and JWKS URL configuration
- **Anonymous** -- `AUTHENTICATION_ANONYMOUS_ACCESS_ENABLED` for development environments [1]

### Authorization

- **AdminList** -- `AUTHORIZATION_ADMINLIST_ENABLED` with admin and read-only user lists
- **RBAC** -- `AUTHORIZATION_RBAC_ENABLED` with role-based access control (mutually exclusive with AdminList) [1]

### Module Configuration

- **`ENABLE_MODULES`** -- Comma-separated list of active modules (vectorizers, generative, rerankers)
- **`API_BASED_MODULES_DISABLED`** -- Restrict external API module access (v1.33+)
- Individual module inference URLs (e.g., `TRANSFORMERS_INFERENCE_API`, `CLIP_INFERENCE_API`) [1]

### Monitoring

- **`PROMETHEUS_MONITORING_ENABLED`** -- Enable Prometheus metrics endpoint
- **`LOG_LEVEL`** -- debug, info, warning, error, fatal, panic
- **`LOG_FORMAT`** -- JSON or text output
- **`QUERY_SLOW_LOG_ENABLED`** and **`QUERY_SLOW_LOG_THRESHOLD`** -- Identify slow queries
- **`DISABLE_TELEMETRY`** -- Opt out of anonymous usage telemetry [1]

### Cluster Settings

- **`CLUSTER_HOSTNAME`** -- Node identifier in multi-node deployments
- **`CLUSTER_JOIN`** -- Founding member address for node discovery
- **`ASYNC_REPLICATION_DISABLED`** -- Toggle asynchronous replication
- **`REPLICATION_MINIMUM_FACTOR`** -- Cluster-wide minimum replication factor [1]

### Resource Limits

- **`DISK_USE_WARNING_PERCENTAGE`** / **`DISK_USE_READONLY_PERCENTAGE`** -- Disk usage thresholds
- **`MEMORY_WARNING_PERCENTAGE`** / **`MEMORY_READONLY_PERCENTAGE`** -- Memory usage thresholds
- **`GRPC_MAX_MESSAGE_SIZE`** -- gRPC payload size limit
- **`MAXIMUM_ALLOWED_COLLECTIONS_COUNT`** -- Collection limit per node (-1 for unlimited) [1]

## Integration Patterns

### With LLM Providers (OpenAI, Anthropic, Cohere, Google)

Weaviate integrates directly with LLM providers through vectorizer and generative modules. Configure a vectorizer module at collection creation to automatically generate embeddings on import. Configure a generative module to enable RAG queries that pass search results to an LLM for augmented responses. API keys are passed via request headers (e.g., `X-OpenAI-Api-Key`, `X-Anthropic-Api-Key`).

### With Agent Frameworks (LangChain, LlamaIndex, Haystack)

Weaviate serves as a vector store backend for agent frameworks. LangChain provides a `Weaviate` vector store class, LlamaIndex offers a `WeaviateVectorStore`, and Haystack has a `WeaviateDocumentStore`. These integrations handle embedding storage, similarity search, and retrieval for RAG pipelines.

### With Embedding Providers (Hugging Face, Ollama, NVIDIA)

For locally hosted inference, connect vectorizer modules to local model servers. Hugging Face Transformers, CLIP, Ollama, and NVIDIA containers can run alongside Weaviate in Docker Compose configurations, providing air-gapped embedding generation.

### With Observability Tools (Prometheus, Grafana)

Enable `PROMETHEUS_MONITORING_ENABLED` to expose metrics at the `/metrics` endpoint. Pair with Grafana dashboards for real-time monitoring of query latency, indexing throughput, and resource utilization.

### With Kubernetes Operators

Deploy production clusters using Helm charts with `values.yaml` configuration. Supports horizontal scaling with sharding and replication, rolling updates, and automated backup schedules.

## Examples

### Create Collection and Import Data

```python
import weaviate
from weaviate.classes.config import Configure
import os

with weaviate.connect_to_weaviate_cloud(
    cluster_url=os.environ["WEAVIATE_URL"],
    auth_credentials=os.environ["WEAVIATE_API_KEY"],
) as client:
    movies = client.collections.create(
        name="Movie",
        vector_config=Configure.Vectors.text2vec_weaviate(),
    )

    data_objects = [
        {"title": "The Matrix", "description": "A computer hacker learns about reality.", "genre": "Science Fiction"},
        {"title": "Spirited Away", "description": "A girl trapped in a spirit world.", "genre": "Animation"},
        {"title": "The Lord of the Rings", "description": "A hobbit on a perilous journey.", "genre": "Fantasy"},
    ]

    movies = client.collections.use("Movie")
    with movies.batch.fixed_size(batch_size=200) as batch:
        for obj in data_objects:
            batch.add_object(properties=obj)
```
[1]

### Semantic Search

```python
movies = client.collections.use("Movie")
response = movies.query.near_text(query="sci-fi", limit=2)

for obj in response.objects:
    print(obj.properties)
```
[1]

### RAG with Generative Search

```python
from weaviate.classes.generate import GenerativeConfig

response = movies.generate.near_text(
    query="sci-fi",
    limit=1,
    grouped_task="Write a tweet with emojis about this movie.",
    generative_provider=GenerativeConfig.anthropic(model="claude-3-5-haiku-latest"),
)
print(response.generative.text)
```
[1]

### Query Agent (Weaviate Cloud)

```python
from weaviate.agents.query import QueryAgent

qa = QueryAgent(client=client, collections=["Movie"])
response = qa.search("Find a cool sci-fi movie.", limit=1)

for obj in response.search_results.objects:
    print(f"Movie: {obj.properties['title']}")
```
[1]

### TypeScript Semantic Search

```typescript
import weaviate, { WeaviateClient, ApiKey, vectors } from 'weaviate-client';

const client: WeaviateClient = await weaviate.connectToWeaviateCloud(
  process.env.WEAVIATE_URL!,
  { authCredentials: new ApiKey(process.env.WEAVIATE_API_KEY!) }
);

const movies = client.collections.get('Movie');
const response = await movies.query.nearText('sci-fi', { limit: 2 });

for (const obj of response.objects) {
  console.log(obj.properties);
}
```
[1]

## Limitations and Considerations

- **No native Windows support**: Weaviate requires containerization (Docker) or Windows Subsystem for Linux (WSL) on Windows platforms
- **Cross-reference performance**: Cross-reference queries may degrade performance at scale; alternative schema designs should be considered for relationship-heavy workloads
- **Flat index scaling**: The flat vector index exhibits linear time complexity and does not scale effectively as datasets grow beyond small collections
- **Dynamic index maturity**: The dynamic index (auto-switching from flat to HNSW) is an experimental feature introduced in v1.25
- **Memory requirements**: HNSW index size in memory is directly proportional to vector count; quantization (PQ, BQ, SQ) mitigates this but introduces precision tradeoffs
- **Async replication consistency**: When write consistency is not set to ALL, writes are asynchronous, meaning temporary inconsistencies can occur across replicas
- **TTL maturity**: Time-to-Live is a technical preview feature (v1.35.0) and may change in future releases
- **Weaviate Agents availability**: Query Agent, Transformation Agent, and Personalization Agent are available only on Weaviate Cloud, not for self-hosted deployments
- **Module dependencies**: API-based vectorizer and generative modules require external API keys and network access to provider endpoints, adding operational dependencies

## Changelog Highlights

- **Weaviate Agents**: Pre-built Query Agent, Transformation Agent (preview), and Personalization Agent (preview) for Weaviate Cloud
- **TTL support (v1.35.0)**: Time-to-Live for automatic object expiration
- **HNSW snapshots (v1.31)**: Persistence snapshots for faster recovery
- **RBAC (v1.29+)**: Role-Based Access Control for fine-grained authorization
- **Async replication (v1.29+)**: Configurable asynchronous replication with tunable frequency
- **Dynamic index (v1.25)**: Experimental auto-switching from flat to HNSW indexes
- **Relative Score Fusion (v1.24)**: New default fusion algorithm for hybrid search
- **Named vectors**: Multiple independent vector spaces per collection with separate configurations
- **Multi-tenancy offloading**: Tenant state management with ACTIVE, INACTIVE, and OFFLOADED states
- **gRPC API**: High-performance Protocol Buffers interface for search and batch operations
- **Weaviate Embeddings**: Managed embedding inference service integrated with Weaviate Cloud

## Citations

- [1] Weaviate Documentation - <https://docs.weaviate.io/weaviate/>
