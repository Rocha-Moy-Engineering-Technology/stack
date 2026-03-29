# Haystack

> Open-source AI orchestration framework by deepset for building production-ready AI agents, multimodal applications, and advanced Retrieval-Augmented Generation (RAG) systems through modular, composable pipelines.

| Field         | Value                                                                |
|---------------|----------------------------------------------------------------------|
| Group         | RAG & Knowledge Retrieval                                            |
| Type          | SDK                                                                  |
| Open Source   | Yes                                                                  |
| GitHub        | [deepset-ai/haystack](https://github.com/deepset-ai/haystack)       |
| Stars         | 24,257                                                               |
| Documentation | [docs.haystack.deepset.ai](https://docs.haystack.deepset.ai/)       |

## Overview

Haystack is an open-source AI orchestration framework developed by deepset, designed for building production-ready AI agents, multimodal applications, and advanced RAG systems. The framework centers on the concept of "context engineering," which refers to the discipline of managing and curating the context that AI systems operate on, including retrieved documents, conversation history, tool outputs, and structured data.

Originally released in 2020, Haystack has evolved from a document search library into a full-featured pipeline orchestration framework. The core abstraction is a directed graph of modular components (Retrievers, Routers, Memory layers, Tools, Evaluators, Generators) that can be tested, swapped, and replaced independently. This modularity allows practitioners to change models, document stores, or processing steps without rewriting application logic.

Haystack supports deployment as REST APIs or Model Context Protocol (MCP) servers through Hayhooks, its companion deployment tool. The framework provides built-in tracing, logging, and evaluation capabilities for production observability.

The ecosystem spans three tiers: the open-source framework (the core library), Enterprise Starter (managed pipelines with support), and Enterprise Platform (full-featured deployment and management infrastructure).

## Core Concepts

- **Pipeline**: A directed acyclic graph of components that defines the flow of data through the system. Pipelines connect components via named inputs and outputs, enabling complex processing workflows with branching, merging, and conditional routing.
- **Component**: The fundamental building block in Haystack. Each component performs a specific task (retrieval, generation, ranking, splitting) and exposes a typed interface with defined inputs and outputs. Components are independently testable and replaceable.
- **Retriever**: A component that fetches relevant documents from a document store based on a query. Haystack provides retrievers for sparse retrieval (BM25), dense retrieval (embedding-based), and hybrid approaches.
- **Generator**: A component that produces text output using a language model. Generators wrap model providers (OpenAI, Anthropic, local models) behind a uniform interface, allowing model swapping without pipeline changes.
- **Router**: A component that directs data flow within a pipeline based on conditions, metadata, or model outputs. Routers enable branching logic such as query classification, fallback strategies, and conditional processing.
- **Document Store**: The storage backend for documents and their metadata. Haystack integrates with multiple backends including Elasticsearch, Weaviate, Pinecone, Qdrant, ChromaDB, and PostgreSQL with pgvector.
- **Memory**: Components that manage conversational context and state across interactions. Memory layers store and retrieve conversation history, enabling multi-turn agent workflows.
- **Tool**: A callable capability that agents can invoke during execution. Tools wrap external functions, APIs, or sub-pipelines, allowing agents to take actions beyond text generation.
- **Evaluator**: A component that scores pipeline outputs against ground truth or quality criteria. Evaluators support metrics such as faithfulness, relevance, and answer correctness for systematic quality assessment.
- **Context Engineering**: The overarching design philosophy in Haystack that treats context management (what information reaches the model, in what form, and when) as a first-class engineering concern rather than an afterthought.

## Architecture

Haystack follows a pipeline-based architecture where components are connected in directed graphs. The architecture separates concerns into distinct layers:

- **Component Layer**: Individual processing units with typed inputs and outputs. Each component declares its interface through decorators, enabling compile-time validation of pipeline connections.
- **Pipeline Layer**: The orchestration graph that connects components. Pipelines handle data routing, parallel execution where possible, and error propagation. A pipeline validates connections at construction time, catching type mismatches before runtime.
- **Document Store Layer**: Abstracted storage backends behind a common interface. Document stores handle indexing, retrieval, and filtering operations. Each backend implements the same protocol, making stores interchangeable.
- **Deployment Layer**: Hayhooks provides REST API and MCP server deployment for pipelines. Pipelines are serialized to YAML and served as endpoints without code changes.

The data flow within a pipeline follows the component graph. Each component receives named inputs, processes them, and produces named outputs that are routed to downstream components. The pipeline runtime manages execution ordering, handles optional inputs, and supports both synchronous and asynchronous execution.

```
Query --> Retriever --> Ranker --> PromptBuilder --> Generator --> Answer
              |                                         ^
              v                                         |
        DocumentStore                              LLM Provider
```

## Key Features

- **Modular Component Design**: Components are self-contained units with typed interfaces. Swap any component (model, retriever, document store) without modifying the rest of the pipeline.
- **Pipeline Serialization**: Pipelines can be serialized to YAML and deserialized back, enabling version control, sharing, and deployment without code changes.
- **Model Agnosticism**: Generators and embedders abstract over model providers. Switch between OpenAI, Anthropic, Cohere, local models, or custom endpoints through configuration.
- **Hybrid Retrieval**: Combine sparse (BM25) and dense (embedding) retrieval strategies in a single pipeline with configurable fusion and ranking.
- **Agent Workflows**: Build autonomous agents that use tools, maintain memory, and make decisions through router-based control flow within pipelines.
- **Evaluation Framework**: Built-in evaluators for RAG quality metrics including faithfulness, answer relevance, context relevance, and semantic similarity.
- **Observability**: Native tracing and logging integration. Pipeline execution traces capture component inputs, outputs, and timing for debugging and monitoring.
- **Deployment via Hayhooks**: Deploy pipelines as REST API endpoints or MCP servers with Hayhooks, enabling integration with existing infrastructure without custom server code.
- **Breaking Change Policies**: Clear versioning and deprecation policies to ensure upgrade paths and production stability.
- **Multimodal Support**: Process and generate across text, images, and structured data within the same pipeline framework.

## Use Cases

- **Retrieval-Augmented Generation**: Build RAG pipelines that retrieve relevant documents from knowledge bases and generate grounded answers. Combine document stores, retrievers, rankers, and generators for end-to-end question answering.
- **Agent Workflows**: Create autonomous agents that reason, plan, and execute multi-step tasks using tools, memory, and conditional routing within pipelines.
- **Text-to-SQL**: Convert natural language questions into SQL queries against structured databases, enabling non-technical users to query data through conversation.
- **Document Processing**: Ingest, clean, split, embed, and index documents from various sources (PDF, HTML, Markdown, DOCX) into vector stores for downstream retrieval.
- **Multimodal Applications**: Build applications that process and reason over combinations of text, images, tables, and structured data within unified pipelines.
- **Conversational Systems**: Develop multi-turn conversational interfaces with memory management, context tracking, and dynamic retrieval based on conversation state.
- **Evaluation and Testing**: Systematically evaluate RAG pipeline quality using built-in metrics, enabling continuous improvement of retrieval and generation performance.

## API Reference Summary

- **`Pipeline`**: The main orchestration class. Methods include `add_component()`, `connect()`, `run()`, and `to_dict()`/`from_dict()` for serialization.
- **`@component`**: Decorator that registers a class as a Haystack component. Components must implement `run()` and declare inputs/outputs via `@component.input_type` and `@component.output_type`.
- **`Document`**: The core data class representing a piece of content with fields for `content`, `meta`, `embedding`, `score`, and `id`.
- **`InMemoryDocumentStore`**: A document store implementation that holds documents in memory. Useful for prototyping and testing.
- **`DocumentWriter`**: A component that writes documents to a document store. Accepts a `document_store` parameter and a `policy` for duplicate handling.
- **`DocumentSplitter`**: A component that splits documents into smaller chunks by sentence, word count, or passage boundaries.
- **`SentenceTransformersDocumentEmbedder`**: Embeds documents using Sentence Transformers models for dense retrieval.
- **`SentenceTransformersTextEmbedder`**: Embeds query text using Sentence Transformers models for matching against document embeddings.
- **`PromptBuilder`**: A component that renders Jinja2 templates into prompts using pipeline variables and retrieved documents.
- **`OpenAIGenerator`**: A generator component that calls OpenAI chat completion APIs. Configurable with model name, parameters, and system prompts.

## Configuration

Haystack pipelines are configured either programmatically or through YAML serialization:

```python
from haystack import Pipeline
from haystack.components.retrievers.in_memory import InMemoryBM25Retriever
from haystack.components.generators import OpenAIGenerator
from haystack.components.builders import PromptBuilder
from haystack.document_stores.in_memory import InMemoryDocumentStore

document_store = InMemoryDocumentStore()

pipeline = Pipeline()
pipeline.add_component("retriever", InMemoryBM25Retriever(document_store=document_store))
pipeline.add_component("prompt_builder", PromptBuilder(
    template="Context: {{documents}} Question: {{query}}"
))
pipeline.add_component("generator", OpenAIGenerator(model="gpt-4o"))

pipeline.connect("retriever.documents", "prompt_builder.documents")
pipeline.connect("prompt_builder.prompt", "generator.prompt")
```

Equivalent YAML configuration:

```yaml
components:
  retriever:
    type: haystack.components.retrievers.in_memory.InMemoryBM25Retriever
    init_parameters:
      document_store:
        type: haystack.document_stores.in_memory.InMemoryDocumentStore
  prompt_builder:
    type: haystack.components.builders.PromptBuilder
    init_parameters:
      template: "Context: {{documents}} Question: {{query}}"
  generator:
    type: haystack.components.generators.OpenAIGenerator
    init_parameters:
      model: gpt-4o
connections:
  - sender: retriever.documents
    receiver: prompt_builder.documents
  - sender: prompt_builder.prompt
    receiver: generator.prompt
```

Environment variables for common providers:

- `OPENAI_API_KEY`: API key for OpenAI models.
- `ANTHROPIC_API_KEY`: API key for Anthropic models.
- `COHERE_API_KEY`: API key for Cohere models.
- `HF_TOKEN`: Hugging Face token for gated models.

## Integration Patterns

- **Document Store Backends**: Haystack integrates with Elasticsearch, OpenSearch, Weaviate, Pinecone, Qdrant, ChromaDB, PostgreSQL (pgvector), Milvus, and others through dedicated integration packages.
- **Model Providers**: Generators and embedders support OpenAI, Anthropic, Cohere, Google AI, Amazon Bedrock, Azure OpenAI, Hugging Face Inference API, and local models via Ollama or vLLM.
- **Hayhooks Deployment**: Serialize pipelines to YAML and deploy them as REST API endpoints or MCP servers using Hayhooks, enabling integration with web applications and AI tool ecosystems.
- **Custom Components**: Create custom components by decorating a class with `@component` and implementing the `run()` method. Custom components integrate seamlessly with built-in components in pipelines.
- **Tracing and Monitoring**: Connect pipeline execution traces to OpenTelemetry-compatible backends (Datadog, Jaeger, Langfuse) for production monitoring.
- **LangChain and LlamaIndex**: While Haystack operates as a standalone framework, documents and embeddings can be shared with other frameworks through common document store backends.

## Examples

Basic RAG pipeline:

```python
from haystack import Pipeline, Document
from haystack.components.retrievers.in_memory import InMemoryBM25Retriever
from haystack.components.generators import OpenAIGenerator
from haystack.components.builders import PromptBuilder
from haystack.document_stores.in_memory import InMemoryDocumentStore

# Index documents
document_store = InMemoryDocumentStore()
documents = [
    Document(content="Haystack is an AI orchestration framework by deepset."),
    Document(content="Haystack supports modular pipelines with retrievers and generators."),
    Document(content="Hayhooks deploys Haystack pipelines as REST APIs."),
]
document_store.write_documents(documents)

# Build RAG pipeline
rag_pipeline = Pipeline()
rag_pipeline.add_component(
    "retriever", InMemoryBM25Retriever(document_store=document_store)
)
rag_pipeline.add_component(
    "prompt_builder",
    PromptBuilder(
        template="Given these documents: {{documents}} Answer: {{query}}"
    ),
)
rag_pipeline.add_component("generator", OpenAIGenerator(model="gpt-4o"))

rag_pipeline.connect("retriever.documents", "prompt_builder.documents")
rag_pipeline.connect("prompt_builder.prompt", "generator.prompt")

# Run the pipeline
result = rag_pipeline.run({
    "retriever": {"query": "What is Haystack?"},
    "prompt_builder": {"query": "What is Haystack?"},
})
print(result["generator"]["replies"][0])
```

Document indexing pipeline:

```python
from haystack import Pipeline
from haystack.components.converters import TextFileToDocument
from haystack.components.preprocessors import DocumentSplitter
from haystack.components.embedders import SentenceTransformersDocumentEmbedder
from haystack.components.writers import DocumentWriter
from haystack.document_stores.in_memory import InMemoryDocumentStore

document_store = InMemoryDocumentStore()

indexing_pipeline = Pipeline()
indexing_pipeline.add_component("converter", TextFileToDocument())
indexing_pipeline.add_component(
    "splitter", DocumentSplitter(split_by="sentence", split_length=3)
)
indexing_pipeline.add_component(
    "embedder", SentenceTransformersDocumentEmbedder()
)
indexing_pipeline.add_component(
    "writer", DocumentWriter(document_store=document_store)
)

indexing_pipeline.connect("converter.documents", "splitter.documents")
indexing_pipeline.connect("splitter.documents", "embedder.documents")
indexing_pipeline.connect("embedder.documents", "writer.documents")

indexing_pipeline.run({"converter": {"sources": ["data/document.txt"]}})
```

## Limitations

- **Python Only**: Haystack is a Python framework with no official SDKs for other languages. Non-Python applications must interact through REST APIs via Hayhooks.
- **Learning Curve for Pipeline Design**: The pipeline graph model, while powerful, requires understanding component interfaces, connection semantics, and data flow patterns that differ from simpler linear API call chains.
- **Integration Package Fragmentation**: Each document store and model provider requires a separate integration package with its own versioning and release cycle, which can lead to dependency management complexity.
- **Memory and State Management**: While Haystack provides memory components for conversational context, complex stateful workflows with branching agent logic may require careful design to avoid state management issues.
- **Ecosystem Maturity Variance**: Core components are well-tested and stable, but community-contributed integrations may vary in maturity, documentation quality, and maintenance status.

## Changelog Highlights

- **Haystack 2.x (2024)**: Complete rewrite with pipeline-as-graph architecture, typed component interfaces, YAML serialization, and breaking API changes from 1.x.
- **Context Engineering Focus**: Expanded emphasis on managing context for AI systems as a core framework concern, with dedicated memory and tool components.
- **MCP Server Support**: Added deployment as MCP servers via Hayhooks, enabling integration with MCP-compatible AI tools and clients.
- **Agent Capabilities**: Introduced agent-oriented components with tool use, routing, and memory for autonomous multi-step workflows.
- **Evaluation Framework**: Built-in RAG evaluation metrics for systematic quality assessment of retrieval and generation pipelines.

## Citations

- [1] Haystack Documentation. https://docs.haystack.deepset.ai/
- [2] Haystack Overview and Introduction. https://haystack.deepset.ai/overview/intro
