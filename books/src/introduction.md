# AI/ML Stack Documentation

This book provides comprehensive documentation for 62 AI/ML tools and platforms organized across 12 functional groups. Each chapter covers the tool's core concepts, installation, architecture, features, use cases, API reference, configuration, integration patterns, and examples -- all sourced from and citing official documentation.

## Catalog Overview

The AI/ML ecosystem is organized into four parts following the dependency chain from foundational infrastructure to specialized integrations:

### Part I: Foundation Models & Infrastructure

The base layer that everything else builds upon.

- **LLM Providers** (3 tools) -- Hosted large language model APIs: OpenAI, Gemini, Claude
- **Inference Serving** (9 tools) -- Engines for hosting and serving model inference: Max, vLLM, SGLang, KServe, Triton, BentoML, Ollama, LM Studio, Hugging Face Transformers
- **GPU Compute & Cloud Platforms** (9 tools) -- Cloud GPU providers and managed AI platforms: Ray, Groq, Cerebras, Modal, RunPod, Vast.ai, Inferless, Vertex AI, AWS Bedrock

### Part II: Application Development

The application layer built on top of foundation models.

- **Agent Frameworks** (8 tools) -- Libraries for building autonomous AI agents: LangChain, LangGraph, AutoGen, CrewAI, ADK, Semantic Kernel, smolagents, Pydantic AI
- **RAG & Knowledge Retrieval** (6 tools) -- Retrieval-Augmented Generation frameworks and vector stores: Haystack, LlamaIndex, pgvector, GraphRAG, Pinecone, Weaviate
- **Structured Output & Prompt Engineering** (4 tools) -- Tools for constraining LLM outputs: DSPy, Outlines, Instructor, BAML

### Part III: Safety, Quality & Operations

Quality assurance, safety, and operational monitoring.

- **Guardrails & Safety** (4 tools) -- Input/output validation and content moderation: Guardrails AI, NeMo Guardrails, OpenAI Moderation, Lakera
- **Evaluation & Testing** (4 tools) -- Frameworks for evaluating LLM application quality: Ragas, DeepEval, OpenAI Evals, promptfoo
- **Observability & LLM Ops** (5 tools) -- Tracing, monitoring, and prompt management: LangSmith, Arize Phoenix, Weights & Biases, Helicone, Langfuse
- **API Gateways & Model Routing** (3 tools) -- Unified API proxies across LLM providers: LiteLLM, Portkey, ccapi

### Part IV: Integration & Data

Connecting AI into broader systems and data pipelines.

- **Workflow Orchestration & Automation** (6 tools) -- Pipeline scheduling and workflow engines: Temporal, Prefect, Airflow, n8n, Activepieces, Node-RED
- **Data Labeling** (1 tool) -- Data annotation and labeling: Label Studio

## How to Use This Book

Each chapter follows a consistent structure:

1. **Overview** -- What the tool is and what problem it solves
2. **Core Concepts** -- Fundamental abstractions and mental models
3. **Installation and Setup** -- Getting started quickly
4. **Architecture** -- Internal structure and extension points
5. **Key Features and Functionality** -- Detailed feature documentation with code examples
6. **Use Cases** -- Step-by-step walkthroughs of common scenarios
7. **API Reference Summary** -- Key classes, functions, and endpoints
8. **Configuration and Customization** -- All configuration options
9. **Integration Patterns** -- How the tool connects with other tools in the catalog
10. **Examples** -- Complete runnable examples
11. **Limitations and Considerations** -- Known constraints and scaling notes
12. **Changelog Highlights** -- Major version milestones
13. **Citations** -- Links to official documentation sources

## Source

All content is derived from official documentation sites. Every claim is backed by a citation linking to the original source. The catalog source of truth is `stacks.csv` in the stack bundle.
