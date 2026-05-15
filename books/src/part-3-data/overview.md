# Part III — Data & Models

This part covers the data and model adaptation layer — how knowledge gets in, how it gets indexed, how it gets searched, and how models get specialized to a domain. Six groups span retrieval orchestration, embedding/reranking, vector storage, ingestion, training, and labeling.

## What's in this part

- **RAG Frameworks** — retrieval-augmented generation orchestration (Haystack, LlamaIndex, GraphRAG)
- **Embeddings & Reranking** — hosted embedding and reranking models (Voyage AI, Cohere, Jina AI)
- **Vector Databases** — vector similarity search engines (pgvector, Pinecone, Weaviate, Qdrant, Milvus)
- **Data Pipelines** — ETL and document ingestion (Unstructured, Airbyte)
- **Fine-tuning** — parameter-efficient training libraries (PEFT, Unsloth, Axolotl)
- **Labeling** — data annotation tooling (Label Studio)

## How to navigate this part

A complete RAG pipeline crosses four of the six groups in this part. **Data Pipelines** (Unstructured for documents, Airbyte for structured data) ingest the source material. **Embeddings & Reranking** (Voyage AI, Cohere, Jina AI) turn that material into vectors and re-score retrieved candidates. **Vector Databases** (pgvector, Pinecone, Weaviate, Qdrant, Milvus) store and search the vectors. **RAG Frameworks** (Haystack, LlamaIndex, GraphRAG) orchestrate the full pipeline from query to answer.

The choice of embedding model often matters more for retrieval quality than the choice of vector database — change `text-embedding-3-small` to `voyage-4` and top-k recall changes more than any prompt rewrite would. The chapters in **Embeddings & Reranking** walk through the trade-offs.

**Fine-tuning** is the parallel track for cases where retrieval isn't enough and the model itself needs domain adaptation. The four chapters (PEFT, Unsloth, Axolotl, plus Hugging Face Transformers in Part I as the foundation) compose into a typical fine-tuning workflow. **Labeling** (Label Studio) addresses the upstream problem of creating training data — relevant both for fine-tuning runs and for building eval datasets.
