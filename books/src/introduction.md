# AI/ML Stack Documentation

This book documents 79 AI/ML tools and platforms organized across 23 functional groups and 4 architectural parts. Each chapter covers the tool's core concepts, architecture, features, use cases, API reference, configuration, integration patterns, and examples — all sourced from and citing official documentation.

## How This Book Is Organized

The catalog follows a dependency-aware structure: foundation infrastructure first, then the application layer built on top of it, then the data/model layer that feeds those applications, and finally the operations layer that runs around all of it. Use the four-part navigation in the sidebar as a mental map of the stack.

### Part I — Foundation & Infrastructure

The base layer everything else builds on: model access (hosted APIs and inference engines), the compute that runs models, the gateways that abstract across providers, and the managed AI platforms that bundle it all together. If you can think of it as "where the model runs and how callers reach it," it's in Part I.

- **LLM Providers** — Hosted large language model APIs (OpenAI, Gemini, Claude)
- **Hosted Inference APIs** — Vendor-hosted inference on custom silicon (Groq, Cerebras)
- **Inference Engines** — Self-hosted serving frameworks and model libraries (Max, vLLM, SGLang, KServe, Triton, BentoML, Hugging Face Transformers)
- **Local Model Runtimes** — Desktop/CLI runners for local development (Ollama, LM Studio)
- **Model Gateways** — Unified proxies and routing across LLM providers (LiteLLM, Portkey, ccapi)
- **Compute & GPU Infrastructure** — Serverless GPU and distributed compute (Ray, Modal, RunPod, Vast.ai, Inferless)
- **Managed AI Platforms** — Cloud end-to-end ML/GenAI platforms (Vertex AI, AWS Bedrock)

### Part II — Application Development

The application layer: the frameworks and protocols developers use to build agents, the runtimes that execute agent code safely, and the building blocks (structured generation, memory) that make agentic applications reliable. If you can think of it as "how an LLM-powered product is composed," it's in Part II.

- **Agent Frameworks** — Libraries for autonomous AI agents (LangChain, LangGraph, AutoGen, CrewAI, ADK, Semantic Kernel, smolagents, Pydantic AI)
- **Agent Protocols** — Open standards for tool and agent interoperability (MCP, A2A)
- **Agent Runtimes & Sandboxes** — Isolated execution environments (E2B)
- **Browser Automation** — Headless browser platforms for web-acting agents (Browserbase)
- **Structured Generation** — Constrained output and schema-typed extraction (DSPy, Outlines, Instructor, BAML)
- **Memory Systems** — Persistent memory and context for agents (Mem0, Zep, Letta)

### Part III — Data & Models

The data and retrieval layer: how knowledge gets in, how it gets indexed, how it gets searched, and how models get adapted to a domain. If you can think of it as "what feeds the model," it's in Part III.

- **RAG Frameworks** — Retrieval-augmented generation orchestration (Haystack, LlamaIndex, GraphRAG)
- **Embeddings & Reranking** — Hosted embedding and reranking models (Voyage AI, Cohere, Jina AI)
- **Vector Databases** — Vector similarity search engines (pgvector, Pinecone, Weaviate, Qdrant, Milvus)
- **Data Pipelines** — ETL and document ingestion (Unstructured, Airbyte)
- **Fine-tuning** — Parameter-efficient fine-tuning (PEFT, Unsloth, Axolotl)
- **Labeling** — Data annotation tooling (Label Studio)

### Part IV — Operations & Quality

The operations layer that surrounds production deployments: input/output safety, evaluation harnesses, observability and tracing, and workflow orchestration. If you can think of it as "what happens before, during, and after a request that isn't the model itself," it's in Part IV.

- **Guardrails** — Input/output validation and content moderation (Guardrails AI, NeMo Guardrails, OpenAI Moderation, Lakera)
- **Evaluation** — LLM/RAG evaluation frameworks (Ragas, DeepEval, OpenAI Evals, promptfoo)
- **Observability** — Tracing, monitoring, prompt management (LangSmith, Arize Phoenix, Weights & Biases, Helicone, Langfuse)
- **Workflow Orchestration** — Pipeline scheduling and automation (Temporal, Prefect, Airflow, n8n, Activepieces, Node-RED)

## How to Use This Book

Each chapter follows a consistent structure:

1. **Overview** — What the tool is and what problem it solves
2. **Core Concepts** — Fundamental abstractions and mental models
3. **Architecture** — Internal structure and extension points
4. **Key Features and Functionality** — Detailed feature documentation with code examples
5. **Use Cases** — Walkthroughs of common scenarios
6. **API Reference Summary** — Key classes, functions, and endpoints
7. **Configuration and Customization** — All configuration options
8. **Integration Patterns** — How the tool connects with other tools in the catalog
9. **Examples** — Complete runnable examples
10. **Limitations and Considerations** — Known constraints and scaling notes
11. **Changelog Highlights** — Major version milestones
12. **Citations** — Links to official documentation sources

## Source

All content is derived from official documentation sites. Every claim is backed by a citation linking to the original source. The catalog's source of truth is `bundles/stack/stacks.csv`; the group enum lives in `bundles/stack/STACKS_SCHEMA.md`.
