# Cohere

> Hosted embedding/reranking/generation models with multilingual support

| Field | Value |
|-------|-------|
| Group | Embeddings & Reranking |
| Type | API/SDK |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.cohere.com/) |

## Overview

Cohere offers hosted models across three product lines that matter for AI applications: the **Command** family (text generation and chat), **Embed** (text embeddings), and **Rerank** (relevance scoring for search). While Cohere does run LLMs (Command A, Command R+, Command R7B, vision and reasoning variants), it is most differentiated in the embeddings-and-reranking layer — Cohere's Rerank API is the de-facto reference reranker in many RAG pipelines and integrates with most vector databases out of the box. [1][2]

Cohere positions itself as an enterprise AI platform with on-platform deployments via AWS Bedrock and Azure AI Foundry alongside the direct Cohere API. The full SDK is available in Python, TypeScript, Go, and Java. [2]

## Core Concepts

### Command (Generation)

The Command family powers text generation and chat. Highlights from the current lineup:

- **command-a-03-2025**: flagship model excelling at tool use, agents, RAG, and multilingual use cases; 256K context, 8K max output. Optimized for high throughput on two-GPU deployments. [3]
- **command-a-reasoning-08-2025**: first Cohere reasoning model, "thinks" before generating; 256K context, 32K max output, 23-language coverage. [3]
- **command-a-vision-07-2025**: multimodal variant for charts, diagrams, OCR, table understanding, document Q&A; 128K context. [3]
- **command-a-translate-08-2025**: state-of-the-art machine translation across 23 languages. [3]
- **command-r-plus-08-2024**, **command-r-08-2024**: previous-generation Command R+ / R models, still widely deployed; both support tool use and RAG. [3]
- **command-r7b-12-2024**: small, fast model for RAG, tool use, agent loops with multi-step reasoning; 128K context. [3]

### Embed (Embeddings)

Cohere's `embed` endpoint converts text (and, for some models, images) into dense vectors for similarity search and RAG. Cohere differentiates Embed by:

- **Multilingual coverage** across 100+ languages in a single embedding space
- **Compression-friendly variants** with configurable embedding sizes
- **Image-text embeddings** for multimodal search

The current generation is the Embed v4 family; consult the models page for the live list. [1][2]

### Rerank

The `rerank` endpoint scores a query against a list of candidate documents and returns relevance scores. Use it after a fast initial retrieval (BM25 or embedding-based) to refine the top-N candidates. Rerank is Cohere's most adopted model in RAG pipelines and is integrated natively in LlamaIndex, Haystack, LangChain, Pinecone, Qdrant, Weaviate, and others. [1]

### Aya (Multilingual)

The Aya family expands multilingual coverage. **Aya Expanse** covers 23 languages; **Aya Vision** is multimodal (image + text in any of those languages). [3]

## Architecture

Cohere's APIs are stateless HTTP endpoints with API-key auth. The primary endpoints are:

- `POST /v1/chat` — Generation, chat, and tool use (Command models)
- `POST /v1/embed` — Text embeddings
- `POST /v1/rerank` — Reranking
- `POST /v1/classify` — Few-shot text classification
- `POST /v1/tokenize`, `/v1/detokenize` — Tokenization helpers

Models are also accessible via AWS Bedrock and Azure AI Foundry with platform-specific model IDs. [2]

## Key Features and Functionality

### Tool Use (Function Calling)

Command models support tool use: pass a list of tool definitions with JSON schemas to `/v1/chat`, and the model returns either a final assistant message or a `tool_calls` array. Execute the tools, return results in the next message, and the model completes its response. Tool use is central to Cohere's agent positioning. [4]

### RAG Mode

The `/v1/chat` endpoint supports a `documents` parameter where you pass retrieved chunks directly with the user's query. The model is trained to ground its response in the supplied documents and emit per-claim citations pointing back to specific chunks. This is a structured alternative to dropping retrieved content into the prompt manually. [4]

### Reranking

The Rerank API takes a query and a list of candidate documents (or a list of structured records with a `rank_fields` selector) and returns relevance scores. The reranker is model-agnostic about how candidates were retrieved — it works with BM25 hits, Cohere embeddings, or any other source.

### Embedding `input_type` Parameter

The embed endpoint accepts an `input_type` parameter (`search_document`, `search_query`, `classification`, `clustering`) so embeddings are tuned for the intended downstream task. Using the correct type at embed-time measurably improves retrieval quality.

### Embed Jobs (Bulk)

For large corpora, the Embed Jobs API runs asynchronous batch embedding jobs against files in cloud storage rather than streaming through the synchronous endpoint. Useful for indexing pipelines that embed millions of documents. [5]

### Citations and Grounding

In RAG mode, the chat response includes structured citations mapping spans of the assistant message back to the supplied documents. This is what makes Cohere's grounded responses verifiable for enterprise use cases like compliance Q&A. [4]

## Use Cases

### Enterprise RAG

The canonical Cohere stack: Embed for indexing, Rerank for refinement, Command (with `documents` parameter) for grounded generation with citations. Each layer is a single API call with sensible defaults.

### Search Quality Upgrade

Drop Rerank into an existing keyword-search system (Elasticsearch, OpenSearch, Vespa, Postgres FTS) as a relevance pass. Cohere is one of the most-cited integrations in vector-DB and search-engine RAG tutorials.

### Multilingual Customer Support

Aya Expanse and Command-A-Translate enable single-stack customer-support assistants that handle queries across 23+ languages without separate per-language models.

### Tool-Using Agents

Command-A and Command-R+ are strong on tool use; combined with the citations/grounding behavior in RAG mode, they suit agent loops that mix retrieval and tool calls.

## API Reference Summary

### /v1/chat — Generation and Chat

```python
response = co.chat(
    model="command-a-03-2025",
    messages=[{"role": "user", "content": "What does a reranker do?"}],
    tools=[...],          # optional, function-calling
    documents=[...],      # optional, RAG mode with grounded citations
)
```

### /v1/embed — Embeddings

```python
response = co.embed(
    texts=["one", "two"],
    model="embed-v4.0",
    input_type="search_document",   # or "search_query"
    embedding_types=["float"],
)
```

### /v1/rerank — Reranking

```python
response = co.rerank(
    query="what is a reranker",
    documents=["a is...", "b is..."],
    model="rerank-v3.5",
    top_n=3,
)
```

## Configuration

### Model Selection

- **Generation**: start with `command-r-08-2024` for cost-balanced general use; move to `command-a-03-2025` for the strongest tool-use/RAG performance; `command-a-reasoning-08-2025` when you need explicit chain-of-thought.
- **Embeddings**: `embed-v4.0` for general English/multilingual; `embed-multilingual-v3.0` for budget-sensitive multilingual; check the live models page for current naming.
- **Reranking**: `rerank-v3.5` is the current default.

### Cloud Deployments

Cohere is available on AWS Bedrock and Azure AI Foundry. Both expose the same model lineup with platform-specific model IDs and IAM-integrated auth, useful for organizations that need data residency or single-vendor billing. [2]

## Integration Patterns

### With Vector Databases (Pinecone, Qdrant, Weaviate, pgvector, Milvus)

All major vector DBs ship Cohere as a first-class embedding option in their quickstart guides.

### With RAG Frameworks (LlamaIndex, Haystack, LangChain)

Cohere's Embed and Rerank classes are pre-built integrations in every major RAG framework, often with `cohere.embed` / `cohere.rerank` modules.

### With Hosted LLMs (Claude, OpenAI, Gemini)

It's common to use Cohere Embed + Rerank for retrieval while delegating final generation to another LLM provider — Cohere's reranker is provider-agnostic.

## Examples

### RAG with Citations

```python
import cohere
co = cohere.ClientV2()

# Retrieved chunks (output of vector search + rerank)
chunks = [
    {"id": "doc-1", "text": "Cohere offers Embed, Rerank, and Command models."},
    {"id": "doc-2", "text": "Rerank scores relevance between a query and candidate documents."},
]

response = co.chat(
    model="command-a-03-2025",
    messages=[{"role": "user", "content": "What is Cohere's Rerank used for?"}],
    documents=chunks,
)

print(response.message.content)
for citation in response.message.citations or []:
    print("cite:", citation.start, citation.end, citation.sources)
```

### Rerank After BM25 Retrieval

```python
candidates = bm25_search(query, top_n=100)  # external
top = co.rerank(
    query=query,
    documents=[c["text"] for c in candidates],
    model="rerank-v3.5",
    top_n=10,
).results
for r in top:
    print(r.relevance_score, candidates[r.index]["id"])
```

## Limitations and Considerations

- **Closed-source models**: No open-weights variant for Command, Embed, or Rerank. The Aya open-weights research models are released separately.
- **Rate limits**: Per-key request and token limits apply; production deployments often use Bedrock or Azure for higher quotas.
- **`input_type` matters**: Same caveat as Voyage — pass `search_document` vs `search_query` correctly or retrieval quality degrades.
- **RAG-mode `documents` shape**: For citations to align, supply each retrieved chunk as a discrete object with a stable `id`. Passing a single concatenated document loses citation granularity.
- **Model lifecycle**: Cohere snapshots model versions in their IDs (`command-a-03-2025`); previous-generation IDs are deprecated on a published timeline — pin versions and watch release notes.

## Changelog Highlights

- **Command A reasoning (08-2025)**: First reasoning model with 256K context, 32K max output, 23-language coverage
- **Command A Vision (07-2025)**: Multimodal Command for charts/diagrams/OCR/document Q&A
- **Command A Translate (08-2025)**: SOTA machine translation across 23 languages
- **Command A (03-2025)**: 256K context, 150% higher throughput than Command R+
- **Rerank v3.5**: Current default reranker
- **Aya Expanse / Aya Vision**: Open-weights multilingual research models from Cohere Labs [3]

## Citations

- [1] Cohere Documentation - <https://docs.cohere.com/>
- [2] Models Overview - <https://docs.cohere.com/docs/models>
- [3] Command Models - <https://docs.cohere.com/docs/command-a>
- [4] Chat API Reference - <https://docs.cohere.com/reference/chat>
- [5] Embed Jobs API - <https://docs.cohere.com/docs/embed-jobs-api>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

Cohere, Command, Embed, Rerank, command-a-03-2025, command-a-reasoning, command-a-vision, command-a-translate, command-r-plus, command-r7b, embed-v4, rerank-v3.5, Aya Expanse, Aya Vision, multilingual, 100+ languages, RAG mode, grounded citations, tool use, function calling, AWS Bedrock, Azure AI Foundry, search_document, search_query, Embed Jobs, classify, tokenize

### Verb-Noun Tasks

- Embed documents with `embed-v4.0` and `input_type=search_document`
- Embed a query with `input_type=search_query`
- Rerank candidate documents with `rerank-v3.5`
- Generate grounded responses with the `documents` parameter
- Emit per-claim citations in RAG mode
- Use tool calling with Command A or Command R+
- Translate across 23 languages with Command A Translate
- Run async batch embedding via the Embed Jobs API
- Deploy Cohere via AWS Bedrock or Azure AI Foundry
- Classify text with few-shot examples
- Build a multilingual customer-support assistant

### User Intent Phrases

- How do I add a reranker to my existing search system?
- How do I produce LLM answers with verifiable per-claim citations?
- Which model is best for multilingual RAG across 100+ languages?
- How do I drop a reranker on top of Elasticsearch BM25 results?
- How do I deploy Cohere through AWS Bedrock for data residency?
- How do I batch-embed millions of documents asynchronously?
- How do I build a tool-using agent with grounded citations?
- Which `input_type` should I use for retrieval queries vs documents?
- How do I get a single API for Embed + Rerank + grounded chat?

### Problem Statements

- My LLM answers lack verifiable citations for compliance
- Search quality drops on multilingual or paraphrased queries
- I need a reranker that drops into LangChain/LlamaIndex without custom glue
- Embedding millions of documents through a sync API is too slow
- I want enterprise deployment via AWS Bedrock or Azure
- My current LLM does not support tool use plus grounded RAG citations
- I cannot get per-claim citations from a raw chat API

### When to Pick This

- Pick this when grounded generation with structured per-claim citations is a first-class requirement (compliance, regulated industries)
- Pick this over Voyage when you also want generation (Command) and reranking under one vendor with citations
- Pick this over Jina AI when enterprise deployments (Bedrock, Azure AI Foundry) and Rerank-as-de-facto-reference matter more than web-to-LLM APIs
- Pick this when Rerank is the most-recommended drop-in across vector DBs and RAG frameworks (LlamaIndex, Haystack, LangChain, Pinecone, Qdrant, Weaviate)
- Pick this when 23-language coverage (Aya Expanse, Command A Translate) is needed in a single stack
- Pick this when you need a hosted RAG-mode chat endpoint, not just an embedding+reranker pair

### Related Terms and Aliases

- Cohere AI
- Cohere API
- Command R+
- Rerank API
- de-facto reference reranker
- Grounded generation
- RAG mode
- Embed v4
- Aya (multilingual)
- ClientV2 (Python SDK)
- Bedrock Cohere
- Azure AI Foundry Cohere
