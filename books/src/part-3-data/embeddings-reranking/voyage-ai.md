# Voyage AI

> Hosted embedding and reranking models with domain-specific tuning for retrieval

| Field | Value |
|-------|-------|
| Group | Embeddings & Reranking |
| Type | API/SDK |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.voyageai.com/) |

## Overview

Voyage AI provides hosted embedding and reranking models targeted at retrieval-quality optimization. Voyage was acquired by MongoDB and its models are positioned as the retrieval-quality layer for RAG and search applications. The product line covers general-purpose embeddings, domain-specific embeddings (code, finance, law), multimodal embeddings, contextualized chunk embeddings, and rerankers. [1]

Embedding models are neural networks (typically transformer-based) that convert text, images, audio, video, or tabular data into dense numerical vectors that capture semantic meaning. These vectors serve as indices for similarity search and RAG. Rerankers are neural networks that output relevance scores between a query and a set of candidate documents; they refine the initial retrieval results from embedding or lexical search (BM25/TF-IDF). [1]

## Core Concepts

### Embedding Models

Voyage's embedding API takes a list of strings and returns a dense vector for each. The current generation is the "voyage-4" series, which is compatible across variants (an embedding produced by `voyage-4-large` is comparable with one from `voyage-4-lite` or `voyage-4-nano`).

Generation-4 model lineup (32,000-token context, configurable output dimensions of 256, 512, 1024 default, or 2048):

- **voyage-4-large**: best general-purpose and multilingual retrieval quality
- **voyage-4**: optimized for general-purpose and multilingual retrieval quality
- **voyage-4-lite**: optimized for latency and cost
- **voyage-4-nano**: open-weight model available on Hugging Face [2]

Domain-specific embeddings:

- **voyage-code-3**: optimized for code retrieval (32,000-token context)
- **voyage-finance-2**: optimized for finance retrieval and RAG
- **voyage-law-2**: optimized for legal retrieval and RAG [2]

### Rerankers

Rerank models take a query and a list of candidate documents and return relevance scores. The typical usage is to retrieve top-100 candidates via embedding search, then rerank to top-10 with the reranker for higher quality at the cost of an extra API round-trip. [1]

### Contextualized Chunk Embeddings

Unlike standard embeddings, where each chunk is embedded independently, contextualized chunk embeddings account for surrounding text in the same document. This improves retrieval quality for documents where a chunk's meaning depends on broader context (e.g., contracts referencing earlier definitions). [3]

### Multimodal Embeddings

Voyage offers multimodal embedding models that take interleaved text and images and produce a unified embedding space across modalities, suitable for image-text retrieval and search over screenshots/diagrams. [4]

### input_type Parameter

The Embeddings API accepts an `input_type` parameter with values `query` or `document`. Using the matching type at embed-time improves retrieval quality compared to embedding both queries and documents identically. [2]

### Flexible Dimensions and Quantization

The voyage-4 series supports configurable output dimensions (256/512/1024/2048) and quantization options for trading storage and inference cost against quality. Embeddings created at different dimensions/precisions remain compatible within the series. [5]

## Architecture

Voyage's APIs are standard HTTP endpoints with API-key auth. Requests are stateless; each call carries the texts (or query/document pairs) plus model and parameters. Responses contain the embeddings or relevance scores plus token counts for usage tracking. The service is available directly from Voyage and also through marketplace integrations (AWS, Azure, Snowflake, Databricks). [1]

## Key Features and Functionality

### Text Embeddings

Single API call: list of strings → list of vectors. Maximum batch size is 1,000 inputs per call. Models support up to 32,000 input tokens depending on variant. [2]

### Reranking

Single API call: query + list of candidate documents → list of relevance scores. Use scores to sort candidates and return the top-k most relevant.

### Tokenization

A `count_tokens` helper lets callers estimate token usage before submission to manage cost and stay within model limits. [7]

### Multilingual Support

The voyage-4 series is optimized for multilingual retrieval, supporting search across documents in different languages with the same model and embedding space. [2]

## Use Cases

### RAG Pipelines

The canonical use case: embed your corpus with `voyage-4` or `voyage-code-3`, store vectors in a vector database (Pinecone, Qdrant, Weaviate, pgvector), embed user queries at request time, retrieve top-k candidates, optionally rerank, then feed retrieved chunks to a generation model. [1]

### Semantic Search

Drop-in replacement for keyword-based search where lexical match underperforms (synonyms, paraphrases, multilingual queries). Embed the corpus offline, query on-line.

### Domain-Specific Retrieval

For specialized corpora (code repositories, financial filings, legal documents), the domain-tuned variants outperform general-purpose models on in-domain benchmarks. [2]

### Hybrid Search

Combine BM25/TF-IDF lexical scores with embedding similarity, then use the reranker as a final quality pass over the merged candidate set.

## API Reference Summary

### Key Endpoints

- `POST /v1/embeddings` — Create embeddings (texts, model, input_type, truncation, output_dimension, output_dtype)
- `POST /v1/rerank` — Rerank candidate documents (query, documents, model, top_k)
- `POST /v1/multimodalembeddings` — Multimodal embeddings (interleaved text and image inputs)

### Python Function Signature (embed)

```python
client.embed(
    texts: list[str],
    model: str,
    input_type: str | None = None,        # "query" or "document"
    truncation: bool | None = None,
    output_dimension: int | None = None,  # 256/512/1024/2048
    output_dtype: str = "float",
)
```

The maximum length of `texts` is 1,000. [2]

## Configuration

### Choosing a Model

- General-purpose multilingual: `voyage-4-large` (best quality) or `voyage-4` (cost-balanced)
- Code: `voyage-code-3`
- Finance: `voyage-finance-2`
- Legal: `voyage-law-2`
- Latency/cost optimized: `voyage-4-lite` or `voyage-4-nano`

### Dimension Selection

Lower dimensions reduce storage and search cost at the price of some retrieval quality. The voyage-4 series is designed so that lower-dimension variants share the embedding space with higher-dimension variants, allowing post-hoc downscaling.

## Integration Patterns

### With Vector Databases (Pinecone, Qdrant, Weaviate, pgvector, Milvus)

Voyage is one of the most commonly recommended embedding providers in vector-DB documentation. Each DB's quickstart typically shows ingestion with a Voyage or OpenAI embedding step.

### With RAG Frameworks (LlamaIndex, Haystack, LangChain)

All major RAG frameworks ship Voyage embedders and rerankers as first-class integrations.

### With MongoDB

Since the MongoDB acquisition, Voyage models are tightly integrated with MongoDB Atlas Vector Search; the docs include dedicated AWS and Azure marketplace listings for the joint product. [1]

## Examples

### Embed and Rerank

```python
import voyageai

client = voyageai.Client()

# 1. Embed documents and queries
docs = ["Embedding models convert text to vectors.", "Rerankers refine candidate lists."]
doc_vectors = client.embed(docs, model="voyage-4", input_type="document").embeddings

query = "What does a reranker do?"
query_vector = client.embed([query], model="voyage-4", input_type="query").embeddings[0]

# 2. (Vector DB step omitted) — assume top-N candidates retrieved
candidates = docs

# 3. Rerank for quality
reranked = client.rerank(query=query, documents=candidates, model="rerank-2", top_k=2)
for r in reranked.results:
    print(r.index, r.relevance_score, candidates[r.index])
```

## Limitations and Considerations

- **Hosted only**: No self-hosted deployment for the closed-source models (`voyage-4-nano` is the open-weight exception, available on Hugging Face).
- **Rate limits**: Per-API-key token-per-minute and request-per-minute limits apply; check current limits in the dashboard.
- **Token counting before submission**: Use the `count_tokens` helper to avoid truncation surprises on long inputs.
- **input_type matters**: Forgetting to pass `query` vs `document` measurably degrades retrieval quality on benchmarks.
- **Reranker cost**: Reranking is a second model call per query — measure end-to-end latency before adopting for latency-sensitive paths.

## Changelog Highlights

- **voyage-4 series (2026)**: New flagship general-purpose embedding family with flexible dimensions (256/512/1024/2048) and `voyage-4-nano` as the open-weight variant
- **voyage-code-3**: Latest code-optimized embedding model
- **Contextualized Chunk Embeddings**: Embeddings that account for in-document context
- **Multimodal Embeddings**: Unified text+image embedding space [1][2]

## Citations

- [1] Introduction - <https://docs.voyageai.com/docs/introduction>
- [2] Text Embeddings - <https://docs.voyageai.com/docs/embeddings>
- [3] Contextualized Chunk Embeddings - <https://docs.voyageai.com/docs/contextualized-chunk-embeddings>
- [4] Multimodal Embeddings - <https://docs.voyageai.com/docs/multimodal-embeddings>
- [5] Flexible Dimensions and Quantization - <https://docs.voyageai.com/docs/flexible-dimensions-and-quantization>
- [6] API Key and Installation - <https://docs.voyageai.com/docs/api-key-and-installation>
- [7] Tokenization - <https://docs.voyageai.com/docs/tokenization>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

Voyage AI, MongoDB, embedding model, reranker, voyage-4, voyage-4-large, voyage-4-lite, voyage-4-nano, voyage-code-3, voyage-finance-2, voyage-law-2, rerank-2, contextualized chunk embeddings, multimodal embeddings, Matryoshka, input_type, query, document, retrieval quality, 32K context, flexible dimensions, quantization, multilingual, MongoDB Atlas Vector Search

### Verb-Noun Tasks

- Embed documents with `voyage-4` for retrieval
- Embed queries with `input_type="query"` for asymmetric retrieval
- Rerank top-100 candidates down to top-10 with the reranker
- Choose `voyage-code-3` for code-base retrieval
- Choose `voyage-finance-2` or `voyage-law-2` for domain corpora
- Configure `output_dimension` (256/512/1024/2048) for storage tradeoffs
- Count tokens before submission with `count_tokens`
- Generate multimodal embeddings for interleaved text and images
- Use contextualized chunk embeddings for documents with intra-document references
- Plug Voyage into MongoDB Atlas Vector Search
- Use the open-weight `voyage-4-nano` from Hugging Face

### User Intent Phrases

- Which embedding model gives the best retrieval quality?
- How do I embed code for a code-search system?
- What is the best embedding model for legal or financial documents?
- How do I improve RAG quality with a reranker?
- How do I shrink embedding dimensions without re-embedding?
- How do I embed images and text in the same space?
- How do I integrate Voyage with MongoDB Atlas?
- Why does `input_type` matter for retrieval quality?
- Where can I self-host a Voyage model?
- How do I batch-embed up to 1,000 texts per call?

### Problem Statements

- General-purpose embeddings underperform on code, finance, or legal corpora
- My retrieval top-k recall is low after a vector-store-only pipeline
- I need to trade storage cost against retrieval quality
- Chunks lose meaning when embedded out of document context
- I cannot self-host most of the leading hosted embedding models
- Forgetting `input_type=query` vs `document` silently degrades retrieval quality
- Reranking adds a second API call to every query

### When to Pick This

- Pick this when retrieval quality is the differentiator and benchmark wins justify hosted API cost
- Pick this over Cohere when domain-specific (code, finance, legal) embeddings or top general-purpose quality matter more than grounded-generation with citations
- Pick this over Jina AI when retrieval quality matters more than web-to-LLM (Reader/Search) ingestion and open-weight availability
- Pick this when Matryoshka-style flexible dimensions (256/512/1024/2048) let you tune storage vs quality post-hoc
- Pick this when you are on MongoDB Atlas Vector Search (tight integration since acquisition)
- Pick `voyage-4-nano` open-weight when you need self-hosted embeddings for compliance

### Related Terms and Aliases

- voyageai (Python package)
- voyage-4 series
- voyage-code-3
- Rerank-2
- Matryoshka representation learning
- input_type=query / input_type=document
- Contextualized chunk embeddings
- Hosted embedding API
- MongoDB Voyage (post-acquisition)
- 32K-token context embeddings
