# Jina AI

> Embedding/reranking/reader-API foundation models for search and web-to-LLM

| Field | Value |
|-------|-------|
| Group | Embeddings & Reranking |
| Type | API/SDK |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.jina.ai/) |

## Overview

Jina AI provides the **Search Foundation** — a family of hosted APIs centered on retrieval rather than generation. The product surface covers embeddings (text, code, multimodal), rerankers (single-vector and ColBERT-style multi-vector), and two web-to-LLM APIs: **Reader** (r.jina.ai) for converting any URL to an LLM-friendly markdown document, and **Search** (s.jina.ai) for SERP-style web search with LLM-ready outputs. [1]

Jina differentiates itself among embedding providers by (1) leading on open-weight model releases (Jina v3 and earlier published openly on Hugging Face), (2) supporting Matryoshka representation learning (one model produces vectors that can be truncated to lower dimensions without re-embedding), and (3) bundling retrieval primitives with the web-content APIs that most RAG pipelines need anyway. [1]

## Core Concepts

### Embeddings API

`POST https://api.jina.ai/v1/embeddings` converts text, images, or code into vectors. Current text models:

- **jina-embeddings-v5-text-small** (677M params, 32K context, 1024d default with Matryoshka truncation to 32/64/128/256/512/1024)
- **jina-embeddings-v5-text-nano** (239M params, 8K context, 768d default with truncation to 32/64/128/256/512/768)
- **jina-embeddings-v4** (3.8B params, multimodal text+image+PDF, 2048d)
- **jina-embeddings-v3** (570M params, multilingual, 1024d)
- **jina-clip-v2** (885M params, multimodal text+image, 1024d)

Code-specific models:
- **jina-code-embeddings-0.5b** and **jina-code-embeddings-1.5b** (Qwen2.5-Coder backbone, NL2Code / Code2Code / Code2NL / Code2Completion task modes) [1]

### Task Parameter

Most Jina models accept a `task` parameter that tunes the embedding for the intended downstream use:

- `retrieval.query` / `retrieval.passage` — different sides of an asymmetric retrieval pair
- `text-matching` — symmetric similarity
- `classification`, `clustering` — for downstream classification or clustering
- `code.query` / `code.passage` (v4) — code retrieval
- `nl2code.query`, `code2code.passage`, etc. (code models) [1]

### Matryoshka Representation Learning

A v5 / v4 / v3 embedding produced at 1024 dimensions can be truncated to 512, 256, 128, etc. without re-embedding, trading quality for storage and search cost. This lets a single index serve different latency/accuracy budgets. [1]

### Reranker API

`POST https://api.jina.ai/v1/rerank` returns relevance scores between a query and candidate documents. Current rerankers:

- **jina-reranker-v3** (0.6B params, multilingual, "last-but-not-late interaction" — causal self-attention between query and documents in one context window)
- **jina-reranker-m0** (2.4B params, multimodal text + image inputs)
- **jina-reranker-v2-base-multilingual** (278M)
- **jina-colbert-v2** (560M, ColBERT-style multi-vector matching) [1]

### Reader API

`POST https://r.jina.ai/` takes a URL and returns its content in markdown (or HTML / text / screenshot). Designed specifically as input for LLMs. The Reader handles JavaScript-heavy sites via a `browser` engine, supports CSS selectors for inclusion/exclusion, can return link/image summaries, and offers `X-Respond-With: readerlm-v2` for a specialized HTML→Markdown LLM. EU-residency endpoints are available at `eu.r.jina.ai`. [1]

### Search API

`POST https://s.jina.ai/` runs a web search and returns LLM-friendly outputs for each result. Adheres to SERP format, supports site-restriction (`X-Site`), country/locale targeting, and the same response-shape options as Reader. EU endpoints at `eu.s.jina.ai`. [1]

### Batch Embeddings API

For bulk processing, `POST https://api.jina.ai/v1/batch/embeddings` accepts inline arrays up to 10,000 items or a GCS URI to a JSONL file up to 50,000 lines. Submit returns a `batch_id`; poll for status (`submitted`/`processing`/`completed`/`failed`/`cancelled`); download as JSONL when ready. [1]

## Architecture

All Jina APIs are simple HTTP endpoints with bearer-token auth (`Authorization: Bearer $JINA_API_KEY`) and `Content-Type: application/json`. Responses are JSON by default; the Reader and Search APIs additionally support `text/event-stream` for streaming. Endpoints are stateless and horizontally scalable; rate limits are per-API-key. [1]

## Key Features and Functionality

### Multimodal Retrieval

`jina-embeddings-v4` and `jina-clip-v2` produce vectors that share an embedding space across text, images, and (for v4) PDFs. This enables cross-modal search — e.g., a text query retrieving relevant images.

### Late Chunking

The `late_chunking: true` parameter on embedding requests concatenates input segments and treats them as a single document during embedding, then chunks the resulting representation — producing chunk-level vectors that retain document-level context. Useful when chunk meaning depends on surrounding text. [1]

### Embedding Output Types

Embeddings can be returned as `float`, `base64`, `binary`, or `ubinary` for tradeoffs between bandwidth and quality. Binary embeddings compress storage and speed up similarity search at modest accuracy cost.

### Reader Engines

The Reader API offers multiple engines: `browser` (full headless render, best quality), `direct` (fast for static HTML), and `cf-browser-rendering` (experimental, JS-heavy sites). Combined with `X-Target-Selector` / `X-Remove-Selector`, this gives fine-grained control over what gets extracted. [1]

### EU Data Residency

Mirror endpoints at `eu.r.jina.ai` and `eu.s.jina.ai` keep all infrastructure and data processing inside the EU. [1]

## Use Cases

### RAG Pipelines

Embed corpus with `jina-embeddings-v5-text-small`, store in a vector DB, rerank with `jina-reranker-v3` for quality, then feed retrieved chunks to a generation model.

### Web-to-LLM

Use the Reader API to convert a URL to clean markdown for the LLM context window. This avoids brittle scraping code and handles client-rendered SPAs.

### Search-Augmented Generation

Use the Search API (s.jina.ai) for live web queries when a user asks about current events. The response is already LLM-shaped — no HTML parsing required.

### Code Search

`jina-code-embeddings-0.5b` / `1.5b` with the appropriate task mode (`nl2code.query` / `code2code.passage`) for code-base search, code review assistants, or technical Q&A.

### Multilingual Retrieval

v5 / v4 / v3 models support 100+ languages in a shared space; useful for global products with mixed-language corpora.

## API Reference Summary

### Endpoints

- `POST https://api.jina.ai/v1/embeddings` — Text/image/code embeddings
- `POST https://api.jina.ai/v1/batch/embeddings` — Bulk async embeddings (and `GET /v1/batch/{id}` for status, `GET /v1/batch/{id}/output` for results, `DELETE /v1/batch/{id}` to cancel)
- `POST https://api.jina.ai/v1/rerank` — Reranking
- `POST https://r.jina.ai/` — Reader (URL → markdown)
- `POST https://s.jina.ai/` — Search (query → SERP)

### Common Parameters

- `model` (required) — e.g., `jina-embeddings-v5-text-small`, `jina-reranker-v3`
- `task` — see Task Parameter section
- `dimensions` — Matryoshka truncation
- `embedding_type` — `float` / `base64` / `binary` / `ubinary`
- `normalized` — L2-normalize output
- `late_chunking` — late-chunking strategy

### Rate Limits

- Embeddings & Reranker: 500 RPM / 1M TPM (Premium: 2k RPM / 5M TPM)
- Reader (r.jina.ai): 500 RPM (Premium: 5k RPM)
- Search (s.jina.ai): 100 RPM (Premium: 1k RPM) [1]

## Configuration

### Choosing a Model

- General multilingual: `jina-embeddings-v5-text-small` (best quality/cost) or `jina-embeddings-v5-text-nano` (edge / low latency)
- Multimodal: `jina-embeddings-v4` (text+image+PDF) or `jina-clip-v2` (text+image)
- Code: `jina-code-embeddings-1.5b` (better) or `0.5b` (cheaper)
- Reranker: `jina-reranker-v3` (default), `jina-reranker-m0` for multimodal, `jina-colbert-v2` if you want ColBERT-style multi-vector

### Choosing a Reader Engine

- `direct` — fast, works for static pages
- `browser` — slower but renders JS-heavy SPAs
- `cf-browser-rendering` — experimental; try for sites the others fail on

## Integration Patterns

### With Vector Databases

Embed with Jina, store in pgvector/Pinecone/Qdrant/Weaviate. Most vector-DB SDKs have a Jina embedder integration.

### With RAG Frameworks

LlamaIndex, Haystack, and LangChain ship Jina embedder/reranker classes.

### Reader → Embeddings Pipeline

A common shape: a user pastes a URL, the Reader API fetches and converts it, the embeddings API indexes the resulting markdown, and the LLM answers questions against that corpus.

### Search-Then-Rerank

Use the Search API for retrieval, then `jina-reranker-v3` to refine the SERP into a tighter top-N list before passing to the LLM.

## Examples

### Embed and Rerank

```bash
# Embed
curl -X POST https://api.jina.ai/v1/embeddings \
  -H "Authorization: Bearer $JINA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "model": "jina-embeddings-v5-text-small",
    "task": "retrieval.passage",
    "input": ["Embeddings convert text to vectors.", "Rerankers refine search results."]
  }'

# Rerank
curl -X POST https://api.jina.ai/v1/rerank \
  -H "Authorization: Bearer $JINA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{
    "model": "jina-reranker-v3",
    "query": "what does a reranker do",
    "documents": ["a is...", "b is..."],
    "top_n": 2
  }'
```

### Reader

```bash
curl -X POST https://r.jina.ai/ \
  -H "Authorization: Bearer $JINA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "X-Engine: browser" \
  -H "X-Return-Format: markdown" \
  -d '{"url": "https://example.com/article"}'
```

### Search

```bash
curl -X POST https://s.jina.ai/ \
  -H "Authorization: Bearer $JINA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"q": "model context protocol overview"}'
```

## Limitations and Considerations

- **No generation**: Jina deliberately does not offer text-generation models; pair with a separate LLM.
- **Rate limits at base tier are restrictive**: production users typically need the premium tier (2k RPM embeddings / 5k RPM Reader).
- **Reader cost on JS-heavy sites**: the `browser` engine is slower and costlier than `direct`; use `direct` first and fall back.
- **Late chunking is opinionated**: it can help or hurt depending on document structure; benchmark before enabling.
- **Multimodal embeddings are model-dependent**: v4 supports PDFs natively; v3 / v5 do not. Plan around the model lineup for your use case.

## Changelog Highlights

- **jina-embeddings-v5-text-small / nano**: Current flagship text embedding family (Qwen3 backbone, Matryoshka, last-token pooling)
- **jina-embeddings-v4**: Multimodal text+image+PDF embeddings, 3.8B params
- **jina-reranker-v3**: 0.6B multilingual reranker with "last-but-not-late interaction" architecture
- **jina-reranker-m0**: 2.4B multimodal reranker
- **Reader v2 + readerlm-v2**: HTML-to-Markdown LLM for cleaner extraction on complex pages
- **EU endpoints**: r.jina.ai / s.jina.ai mirrors for EU data residency [1]

## Citations

- [1] Jina AI Search Foundation Meta-Prompt - <https://docs.jina.ai/>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

Jina AI, Search Foundation, Reader, Search API, r.jina.ai, s.jina.ai, jina-embeddings-v5, jina-embeddings-v4, jina-embeddings-v3, jina-clip-v2, jina-code-embeddings, jina-reranker-v3, jina-reranker-m0, jina-colbert-v2, Matryoshka, late chunking, multimodal, multilingual, web-to-LLM, URL to markdown, SERP, readerlm-v2, open weights, Hugging Face, EU data residency, batch embeddings, last-but-not-late interaction

### Verb-Noun Tasks

- Convert a URL to LLM-ready markdown via Reader (r.jina.ai)
- Run a SERP-style web search with s.jina.ai
- Embed text with `jina-embeddings-v5-text-small`
- Truncate embeddings via Matryoshka dimensions (32/64/128/256/512/1024)
- Rerank candidates with `jina-reranker-v3`
- Rerank multimodal (text+image) pairs with `jina-reranker-m0`
- Use `jina-colbert-v2` for ColBERT multi-vector matching
- Embed code with `jina-code-embeddings-1.5b` and `nl2code.query` task
- Enable late chunking for context-aware chunk vectors
- Submit a batch embedding job (up to 50,000 lines)
- Use `cf-browser-rendering` for JS-heavy site extraction
- Route traffic through EU endpoints for data residency

### User Intent Phrases

- How do I scrape a JS-heavy URL into clean markdown for an LLM?
- How do I run a web search and feed results to an LLM?
- Which open-weight embedding model can I self-host?
- How do I truncate embedding dimensions without re-embedding?
- How do I embed images and text in the same space?
- How do I do code-base search with embeddings?
- How do I keep embedding infrastructure inside the EU?
- How do I batch-embed 50,000 documents asynchronously?
- How do I add a reranker that handles images too?

### Problem Statements

- Scraping client-rendered SPAs into LLM context is brittle and slow
- Searching the live web from an agent returns raw HTML, not LLM-ready text
- Most hosted embedding providers do not publish open weights
- One embedding size locks me into a single storage/quality tradeoff
- I need multimodal retrieval over text + images + PDFs
- Code retrieval with generic embeddings underperforms
- EU customers need data processing inside the EU

### When to Pick This

- Pick this when web-to-LLM ingestion (Reader, Search) is part of the same pipeline as embeddings and reranking
- Pick this over Voyage when open-weight availability (v3, v4-nano on Hugging Face) and multimodal CLIP-style retrieval matter
- Pick this over Cohere when web-content APIs and ColBERT-style multi-vector reranking matter more than grounded-generation citations
- Pick this when long-context embeddings (32K, with last-token pooling) and Matryoshka truncation are core requirements
- Pick this when EU data residency endpoints (eu.r.jina.ai, eu.s.jina.ai) are needed
- Pick this when late chunking would help your in-document context dependencies

### Related Terms and Aliases

- Jina Search Foundation
- Reader API (URL → markdown)
- Search API (query → SERP)
- jina-embeddings (v3/v4/v5)
- jina-reranker
- jina-clip-v2
- ColBERT (jina-colbert-v2)
- readerlm-v2
- Matryoshka representation learning
- Late chunking
- Multimodal embeddings
- Open-weight embedding models
