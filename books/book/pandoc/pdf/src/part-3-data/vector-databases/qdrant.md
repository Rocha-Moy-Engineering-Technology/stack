[Header 1 ("qdrant", [], []) [Str "Qdrant"], BlockQuote [Para [Str "Open-source AI-native vector database written in Rust for high-performance similarity search at scale"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Qdrant"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Vector Databases"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK/Infra"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "qdrant/qdrant"] ("https://github.com/qdrant/qdrant", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "29275"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "qdrant.tech/documentation"] ("https://qdrant.tech/documentation/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Qdrant (pronounced \"quadrant\") is an open-source vector database and semantic search engine written in Rust. It stores, indexes, and searches high-dimensional vector embeddings with associated metadata payloads, enabling similarity search that goes beyond keyword matching. Founded in 2021 and based in Berlin, Qdrant provides fast, scalable vector similarity search with convenient APIs ", Str "[", Str "1", Str "]", Str "."], Para [Str "The platform converts unstructured data (text, images, audio) into dense vector embeddings using embedding models, mapping them into high-dimensional space where semantically similar items cluster together. Qdrant combines dense vectors for contextual understanding with sparse vectors for precise lexical keyword matching through hybrid retrieval ", Str "[", Str "1", Str "]", Str "."], Para [Str "Qdrant offers four deployment options ", Str "[", Str "1", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Open Source (Self-Hosted)"], Str ": Full control over data and deployment with Docker, Kubernetes, or binary"]], [Plain [Strong [Str "Qdrant Cloud (Managed)"], Str ": Fully managed service with high availability and zero-downtime upgrades"]], [Plain [Strong [Str "Hybrid Cloud"], Str ": Managed control plane with data remaining in the user's infrastructure"]], [Plain [Strong [Str "Private Cloud"], Str ": Fully isolated deployment on user's own infrastructure"]]], Para [Str "Official client libraries are available for Python, JavaScript/TypeScript, Rust, Go, Java, and .NET, with both REST (port 6333) and gRPC (port 6334) API interfaces ", Str "[", Str "5", Str "]", Str "."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("collections", ["unnumbered", "unlisted"], []) [Str "Collections"], Para [Str "A ", Strong [Str "collection"], Str " is a named set of points (vectors with payloads) among which you search. All vectors within a collection must share the same dimensionality and distance metric, though named vectors allow multiple vector types per point with independent configurations. Collections support four distance metrics: Dot product, Cosine similarity, Euclidean distance, and Manhattan distance ", Str "[", Str "2", Str "]", Str "."], Header 3 ("points", ["unnumbered", "unlisted"], []) [Str "Points"], Para [Str "A ", Strong [Str "point"], Str " is the central entity consisting of three components: an identifier (64-bit unsigned integer or UUID), a vector (dense, sparse, or multi-vector), and an optional payload (metadata). Points are the fundamental storage unit combining vector embeddings with arbitrary JSON metadata ", Str "[", Str "3", Str "]", Str "."], Header 3 ("vectors", ["unnumbered", "unlisted"], []) [Str "Vectors"], Para [Str "Qdrant supports multiple vector types ", Str "[", Str "3", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Dense vectors"], Str ": Standard floating-point embeddings (Float32, Uint8) from neural network models"]], [Plain [Strong [Str "Sparse vectors"], Str ": High-dimensional vectors with mostly zero values, represented as index-value pairs for keyword-based search (BM25-style)"]], [Plain [Strong [Str "Multi-vectors"], Str ": Matrices from late-interaction models like ColBERT, where each document is represented as multiple vector chunks"]], [Plain [Strong [Str "Named vectors"], Str ": Multiple independently configured vectors per point, enabling storage of different embedding types (e.g., image and text) in a single collection"]]], Header 3 ("payloads", ["unnumbered", "unlisted"], []) [Str "Payloads"], Para [Strong [Str "Payloads"], Str " are arbitrary JSON metadata attached to points. Qdrant supports payload types including keyword (string), integer, float, bool, geo coordinates, datetime, text (full-text indexed), and UUID. Payload indexes extend the HNSW graph, enabling filtering criteria during the semantic search phase in a single-pass traversal rather than separate pre/post-filtering steps ", Str "[", Str "1", Str "]", Str "[", Str "4", Str "]", Str "."], Header 3 ("segments", ["unnumbered", "unlisted"], []) [Str "Segments"], Para [Str "Collections organize data into ", Strong [Str "segments"], Str ", each with independent vector storage, payload storage, indexes, and an ID mapper. Segments are either appendable (full CRUD) or non-appendable (read and delete only). Data integrity is maintained through a Write-Ahead Log (WAL) that orders operations sequentially before propagating to segments ", Str "[", Str "8", Str "]", Str "."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Qdrant uses a client-server architecture with distributed clustering capabilities:"], CodeBlock ("", [""], []) "┌─────────────────────────────────────────────────┐
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
", Para [Str "In distributed mode, Qdrant uses ", Strong [Str "Raft consensus"], Str " for cluster topology and collection structure operations. Point operations (insert, search) bypass consensus for low overhead. Collections split into ", Strong [Str "shards"], Str " distributed across nodes via consistent hashing. ", Strong [Str "Replication"], Str " creates shard copies across nodes, with configurable write consistency factors and read consistency levels (all, majority, quorum) ", Str "[", Str "6", Str "]", Str "."], Header 3 ("vector-storage-options", ["unnumbered", "unlisted"], []) [Str "Vector Storage Options"], BulletList [[Plain [Strong [Str "In-Memory"], Str ": All vectors in RAM for maximum speed; disk used only for persistence"]], [Plain [Strong [Str "Memmap"], Str ": Memory-mapped files using page cache for near in-memory performance with lower RAM usage"]], [Plain [Strong [Str "On-Disk"], Str ": Full disk-based storage for datasets exceeding available memory ", Str "[", Str "8", Str "]"]]], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "HNSW Index"], Str ": Hierarchical Navigable Small World graph for fast approximate nearest neighbor search with configurable m, ef_construct, and ef parameters"]], [Plain [Strong [Str "Filterable HNSW"], Str ": Payload indexes extend HNSW graph edges, enabling single-pass filtered vector search without separate pre/post-filtering"]], [Plain [Strong [Str "Hybrid Search"], Str ": Combine dense and sparse vector queries with Reciprocal Rank Fusion (RRF) or Distribution-Based Score Fusion (DBSF) via the prefetch mechanism"]], [Plain [Strong [Str "Multi-Stage Search"], Str ": Nested prefetch pipelines for re-scoring — retrieve candidates with compact vectors, then re-rank with full-precision or ColBERT multi-vectors"]], [Plain [Strong [Str "Quantization"], Str ": Scalar (4x compression), Binary (up to 32x compression, 40x speedup), and Product quantization (up to 64x compression) with configurable rescoring and oversampling"]], [Plain [Strong [Str "Named Vectors"], Str ": Store multiple independently configured vector types per point (e.g., image and text embeddings)"]], [Plain [Strong [Str "Sparse Vectors"], Str ": Native sparse vector support with IDF modifier for keyword-based search"]], [Plain [Strong [Str "Rich Filtering"], Str ": Boolean clauses (must, should, must_not), range, geo (bounding box, radius, polygon), full-text match, datetime, nested object filters"]], [Plain [Strong [Str "Distributed Clustering"], Str ": Sharding with consistent hashing, replication, Raft consensus, and three shard transfer methods (stream_records, snapshot, wal_delta)"]], [Plain [Strong [Str "Collection Aliases"], Str ": Zero-downtime model upgrades by atomically switching collection pointers"]], [Plain [Strong [Str "ACORN Search"], Str ": Enhanced HNSW exploration for restrictive multi-filter queries via second-hop neighbor traversal"]], [Plain [Strong [Str "Grouping API"], Str ": Aggregate search results by payload field to avoid redundant items"]], [Plain [Strong [Str "Batch Operations"], Str ": Execute multiple operations (upsert, delete, update vectors, set payload) in a single request"]], [Plain [Strong [Str "Conditional Updates"], Str ": Optimistic locking with version-based filters to prevent concurrent overwrites"]], [Plain [Strong [Str "FastEmbed"], Str ": Built-in embedding generation library for client-side inference"]], [Plain [Strong [Str "Web UI Dashboard"], Str ": Built-in web interface for collection management and query exploration"]], [Plain [Strong [Str "MCP Server"], Str ": Model Context Protocol server for AI assistant integration"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Semantic Search"], Str ": Find documents, products, or media by meaning rather than exact keywords using dense vector similarity"]], [Plain [Strong [Str "Retrieval-Augmented Generation (RAG)"], Str ": Store document chunk embeddings and retrieve relevant context for LLM prompts with hybrid dense+sparse search"]], [Plain [Strong [Str "Recommendation Systems"], Str ": Find similar items based on user behavior embeddings with payload-based filtering for business rules"]], [Plain [Strong [Str "Image and Video Search"], Str ": Index visual embeddings for reverse image search and content-based retrieval"]], [Plain [Strong [Str "Anomaly Detection"], Str ": Identify outliers by measuring vector distances from normal behavior patterns"]], [Plain [Strong [Str "Multi-Tenant Applications"], Str ": Serve millions of users with payload-based partitioning or user-defined custom sharding for strict isolation"]], [Plain [Strong [Str "E-Commerce"], Str ": Product search combining visual similarity, text descriptions, and metadata filtering (price, category, availability)"]], [Plain [Strong [Str "Content Deduplication"], Str ": Detect near-duplicate documents or media using vector similarity thresholds"]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("collection-operations", ["unnumbered", "unlisted"], []) [Str "Collection Operations"], CodeBlock ("", ["python"], []) "from qdrant_client import QdrantClient, models

client = QdrantClient(url=\"http://localhost:6333\")

# Create collection
client.create_collection(
    collection_name=\"my_collection\",
    vectors_config=models.VectorParams(size=768, distance=models.Distance.COSINE),
)

# Multi-vector collection
client.create_collection(
    collection_name=\"multi_vec\",
    vectors_config={
        \"image\": models.VectorParams(size=512, distance=models.Distance.DOT),
        \"text\": models.VectorParams(size=768, distance=models.Distance.COSINE),
    },
)

# Check existence
client.collection_exists(\"my_collection\")

# Get collection info
client.get_collection(\"my_collection\")
", Header 3 ("upserting-points", ["unnumbered", "unlisted"], []) [Str "Upserting Points"], CodeBlock ("", ["python"], []) "client.upsert(
    collection_name=\"my_collection\",
    wait=True,
    points=[
        models.PointStruct(
            id=1,
            vector=[0.05, 0.61, 0.76, 0.74],
            payload={\"city\": \"Berlin\", \"category\": \"travel\"},
        ),
        models.PointStruct(
            id=2,
            vector=[0.19, 0.81, 0.75, 0.11],
            payload={\"city\": \"London\", \"category\": \"business\"},
        ),
    ],
)
", Header 3 ("vector-search", ["unnumbered", "unlisted"], []) [Str "Vector Search"], CodeBlock ("", ["python"], []) "# Basic search
results = client.query_points(
    collection_name=\"my_collection\",
    query=[0.2, 0.1, 0.9, 0.7],
    limit=5,
    with_payload=True,
)

# Filtered search
results = client.query_points(
    collection_name=\"my_collection\",
    query=[0.2, 0.1, 0.9, 0.7],
    query_filter=models.Filter(
        must=[
            models.FieldCondition(
                key=\"city\",
                match=models.MatchValue(value=\"London\"),
            )
        ]
    ),
    limit=5,
)

# Search with parameters
results = client.query_points(
    collection_name=\"my_collection\",
    query=[0.2, 0.1, 0.9, 0.7],
    search_params=models.SearchParams(hnsw_ef=128, exact=False),
    limit=10,
)
", Header 3 ("hybrid-search-dense--sparse", ["unnumbered", "unlisted"], []) [Str "Hybrid Search (Dense + Sparse)"], CodeBlock ("", ["python"], []) "results = client.query_points(
    collection_name=\"my_collection\",
    prefetch=[
        models.Prefetch(
            query=models.SparseVector(indices=[1, 42], values=[0.22, 0.8]),
            using=\"sparse\",
            limit=20,
        ),
        models.Prefetch(
            query=[0.01, 0.45, 0.67],
            using=\"dense\",
            limit=20,
        ),
    ],
    query=models.RrfQuery(rrf=models.Rrf(weights=[3.0, 1.0])),
)
", Header 3 ("payload-filtering", ["unnumbered", "unlisted"], []) [Str "Payload Filtering"], CodeBlock ("", ["python"], []) "# Scroll with complex filter
results = client.scroll(
    collection_name=\"my_collection\",
    scroll_filter=models.Filter(
        must=[
            models.FieldCondition(key=\"city\", match=models.MatchValue(value=\"Berlin\")),
        ],
        must_not=[
            models.FieldCondition(key=\"category\", match=models.MatchValue(value=\"spam\")),
        ],
    ),
    limit=10,
    with_payload=True,
)

# Range filter
models.FieldCondition(key=\"price\", range=models.Range(gte=100.0, lte=450.0))

# Geo filter
models.FieldCondition(
    key=\"location\",
    geo_radius=models.GeoRadius(
        center=models.GeoPoint(lon=13.403683, lat=52.520711),
        radius=1000.0,
    ),
)
", Header 3 ("delete-operations", ["unnumbered", "unlisted"], []) [Str "Delete Operations"], CodeBlock ("", ["python"], []) "# Delete by IDs
client.delete(
    collection_name=\"my_collection\",
    points_selector=models.PointIdsList(points=[0, 3, 100]),
)

# Delete by filter
client.delete(
    collection_name=\"my_collection\",
    points_selector=models.FilterSelector(
        filter=models.Filter(
            must=[models.FieldCondition(key=\"city\", match=models.MatchValue(value=\"London\"))]
        )
    ),
)
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("collection-level-configuration", ["unnumbered", "unlisted"], []) [Str "Collection-Level Configuration"], CodeBlock ("", ["python"], []) "client.create_collection(
    collection_name=\"optimized\",
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
", Header 3 ("payload-index-configuration", ["unnumbered", "unlisted"], []) [Str "Payload Index Configuration"], CodeBlock ("", ["python"], []) "# Keyword index
client.create_payload_index(
    collection_name=\"my_collection\",
    field_name=\"category\",
    field_schema=models.PayloadSchemaType.KEYWORD,
)

# Full-text index with tokenization
client.create_payload_index(
    collection_name=\"my_collection\",
    field_name=\"description\",
    field_schema=models.TextIndexParams(
        type=\"text\",
        tokenizer=models.TokenizerType.WORD,
        min_token_len=2,
        max_token_len=15,
        lowercase=True,
    ),
)

# Tenant-optimized index
client.create_payload_index(
    collection_name=\"my_collection\",
    field_name=\"tenant_id\",
    field_schema=models.KeywordIndexParams(
        type=\"keyword\",
        is_tenant=True,
    ),
)
", Header 3 ("server-configuration-yaml", ["unnumbered", "unlisted"], []) [Str "Server Configuration (YAML)"], Para [Str "Key ", Code ("", [], []) "config.yaml", Str " parameters:"], CodeBlock ("", ["yaml"], []) "storage:
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
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("fastembed-built-in-embeddings", ["unnumbered", "unlisted"], []) [Str "FastEmbed (Built-in Embeddings)"], Para [Str "Qdrant maintains FastEmbed, a lightweight embedding generation library. Install with ", Code ("", [], []) "pip install qdrant-client[fastembed]", Str " for client-side inference without external API calls ", Str "[", Str "1", Str "]", Str "."], Header 3 ("langchain-integration", ["unnumbered", "unlisted"], []) [Str "LangChain Integration"], Para [Str "Qdrant provides a native LangChain vector store for RAG pipelines, enabling document embedding storage and retrieval during LLM prompt construction."], Header 3 ("llamaindex-integration", ["unnumbered", "unlisted"], []) [Str "LlamaIndex Integration"], Para [Str "LlamaIndex supports Qdrant as a vector store backend for document indexing and retrieval in RAG applications."], Header 3 ("mcp-server", ["unnumbered", "unlisted"], []) [Str "MCP Server"], Para [Str "Qdrant provides a Model Context Protocol server (", Code ("", [], []) "mcp-server-qdrant", Str ") for connecting AI coding assistants and agents directly to Qdrant collections ", Str "[", Str "1", Str "]", Str "."], Header 3 ("qdrant-edge", ["unnumbered", "unlisted"], []) [Str "Qdrant Edge"], Para [Str "A lightweight deployment mode for edge computing environments with resource constraints ", Str "[", Str "1", Str "]", Str "."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("rag-pipeline-with-hybrid-search", ["unnumbered", "unlisted"], []) [Str "RAG Pipeline with Hybrid Search"], CodeBlock ("", ["python"], []) "from qdrant_client import QdrantClient, models

client = QdrantClient(url=\"http://localhost:6333\")

# Create collection with dense + sparse vectors
client.create_collection(
    collection_name=\"rag_docs\",
    vectors_config={
        \"dense\": models.VectorParams(size=768, distance=models.Distance.COSINE),
    },
    sparse_vectors_config={
        \"sparse\": models.SparseVectorParams(),
    },
)

# Insert document chunks
client.upsert(
    collection_name=\"rag_docs\",
    points=[
        models.PointStruct(
            id=1,
            vector={
                \"dense\": [0.1, 0.2, ...],  # from embedding model
                \"sparse\": models.SparseVector(indices=[5, 10, 42], values=[0.5, 0.3, 0.8]),
            },
            payload={\"text\": \"Document chunk content\", \"source\": \"docs/intro.md\"},
        ),
    ],
)

# Hybrid search combining semantic + keyword
results = client.query_points(
    collection_name=\"rag_docs\",
    prefetch=[
        models.Prefetch(query=[0.1, 0.2, ...], using=\"dense\", limit=20),
        models.Prefetch(
            query=models.SparseVector(indices=[5, 42], values=[0.5, 0.8]),
            using=\"sparse\",
            limit=20,
        ),
    ],
    query=models.RrfQuery(rrf=models.Rrf()),
    with_payload=True,
    limit=5,
)
", Header 3 ("multi-tenant-collection", ["unnumbered", "unlisted"], []) [Str "Multi-Tenant Collection"], CodeBlock ("", ["python"], []) "# Create collection with tenant optimization
client.create_collection(
    collection_name=\"multi_tenant\",
    vectors_config=models.VectorParams(size=768, distance=models.Distance.COSINE),
)

# Create tenant-optimized index
client.create_payload_index(
    collection_name=\"multi_tenant\",
    field_name=\"tenant_id\",
    field_schema=models.KeywordIndexParams(type=\"keyword\", is_tenant=True),
)

# Insert tenant-specific data
client.upsert(
    collection_name=\"multi_tenant\",
    points=[
        models.PointStruct(
            id=1,
            vector=[0.1, 0.2, ...],
            payload={\"tenant_id\": \"company_a\", \"text\": \"Tenant A document\"},
        ),
    ],
)

# Search scoped to tenant
results = client.query_points(
    collection_name=\"multi_tenant\",
    query=[0.1, 0.2, ...],
    query_filter=models.Filter(
        must=[models.FieldCondition(key=\"tenant_id\", match=models.MatchValue(value=\"company_a\"))]
    ),
    limit=10,
)
", Header 3 ("quantized-collection-for-large-scale-deployment", ["unnumbered", "unlisted"], []) [Str "Quantized Collection for Large-Scale Deployment"], CodeBlock ("", ["python"], []) "# Binary quantization for high-dimensional embeddings (32x compression)
client.create_collection(
    collection_name=\"large_scale\",
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
    collection_name=\"large_scale\",
    query=[0.1, 0.2, ...],
    search_params=models.SearchParams(
        quantization=models.QuantizationSearchParams(
            rescore=True,
            oversampling=2.0,
        ),
    ),
    limit=10,
)
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "No ACID transactions"], Str ": Point operations bypass Raft consensus for performance; distributed atomicity is limited to single-point operations"]], [Plain [Strong [Str "Single distance metric per vector"], Str ": All dense vectors within a collection (or named vector) must use the same distance metric and dimensionality"]], [Plain [Strong [Str "Sparse vector constraints"], Str ": Sparse vectors support only dot-product distance and always use exact matching (no approximate indexing)"]], [Plain [Strong [Str "Two-node cluster limitations"], Str ": Collection create/edit/delete operations fail when one node is offline since recovery requires >50% of nodes healthy ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "No built-in embedding"], Str ": Qdrant stores and searches vectors but does not generate embeddings (FastEmbed is a separate client-side library); server-side inference is limited to Qdrant Cloud"]], [Plain [Strong [Str "Product quantization performance"], Str ": PQ is slower than scalar quantization (non-SIMD-friendly) with significant accuracy loss (", Str "~", Str "0.7 accuracy) ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Memory requirements"], Str ": In-memory HNSW indexes require substantial RAM for large datasets; memmap or on-disk storage must be explicitly configured"]], [Plain [Strong [Str "Default no authentication"], Str ": Qdrant starts with no encryption or authentication by default; security must be explicitly configured ", Str "[", Str "9", Str "]"]], [Plain [Strong [Str "Eventual consistency default"], Str ": Distributed deployments prioritize availability and throughput; strong consistency requires explicit configuration of write ordering and read consistency levels ", Str "[", Str "6", Str "]"]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "v1.17.0"], Str ": RRF weighted fusion, update modes (insert_only, update_only), disable HNSW edges per field"]], [Plain [Strong [Str "v1.16.0"], Str ": ACORN search algorithm, conditional updates with filters, collection metadata, ASCII folding in text indexes, full-text any matching"]], [Plain [Strong [Str "v1.15.0"], Str ": 1.5-bit and 2-bit binary quantization, asymmetric quantization with query encoding"]], [Plain [Strong [Str "v1.13.0"], Str ": Resharding (Cloud), has_vector filter condition"]], [Plain [Strong [Str "v1.11.0"], Str ": Distribution-Based Score Fusion (DBSF), on-disk payload indexes, tenant/principal index types, UUID matching, grouping in hybrid queries"]], [Plain [Strong [Str "v1.10.0"], Str ": Sparse vector IDF modifier"]], [Plain [Strong [Str "v1.9.0"], Str ": Uint8 vector datatype support"]], [Plain [Strong [Str "v1.8.0"], Str ": WAL delta shard transfer, datetime range filters, order-by payload scrolling, parameterized integer indexes"]], [Plain [Strong [Str "v1.7.0"], Str ": Sparse vector support, user-defined custom sharding, snapshot shard transfer"]], [Plain [Strong [Str "v1.5.0"], Str ": Binary quantization, batch update operations"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Overview - https://qdrant.tech/documentation/overview/"]], [Plain [Str "[", Str "2", Str "]", Str " Collections - https://qdrant.tech/documentation/concepts/collections/"]], [Plain [Str "[", Str "3", Str "]", Str " Points - https://qdrant.tech/documentation/concepts/points/"]], [Plain [Str "[", Str "4", Str "]", Str " Filtering - https://qdrant.tech/documentation/concepts/filtering/"]], [Plain [Str "[", Str "5", Str "]", Str " API & SDKs - https://qdrant.tech/documentation/interfaces/"]], [Plain [Str "[", Str "6", Str "]", Str " Distributed Deployment - https://qdrant.tech/documentation/guides/distributed_deployment/"]], [Plain [Str "[", Str "7", Str "]", Str " Quantization - https://qdrant.tech/documentation/guides/quantization/"]], [Plain [Str "[", Str "8", Str "]", Str " Storage - https://qdrant.tech/documentation/concepts/storage/"]], [Plain [Str "[", Str "9", Str "]", Str " Quickstart - https://qdrant.tech/documentation/quickstart/"]], [Plain [Str "[", Str "10", Str "]", Str " Search - https://qdrant.tech/documentation/concepts/search/"]], [Plain [Str "[", Str "11", Str "]", Str " Hybrid Queries - https://qdrant.tech/documentation/concepts/hybrid-queries/"]], [Plain [Str "[", Str "12", Str "]", Str " Indexing - https://qdrant.tech/documentation/concepts/indexing/"]]]]