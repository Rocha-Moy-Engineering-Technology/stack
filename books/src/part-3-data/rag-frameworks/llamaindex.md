# LlamaIndex

> Leading framework for building LLM-powered agents and workflows over your data, providing context augmentation that makes private and domain-specific data available to large language models.

| Field         | Value                                                                 |
|---------------|-----------------------------------------------------------------------|
| Name          | LlamaIndex                                                            |
| Group         | RAG Frameworks                                             |
| Type          | SDK                                                                   |
| Open Source   | Yes                                                                   |
| GitHub        | [run-llama/llama_index](https://github.com/run-llama/llama_index)     |
| Stars         | 49412                                                                |
| Documentation | [developers.llamaindex.ai](https://developers.llamaindex.ai/python/framework/) |

## Overview

LlamaIndex is a data framework designed to connect large language models with external data sources. At its core, LlamaIndex solves the context augmentation problem: making private, domain-specific, or otherwise inaccessible data available to LLMs so they can reason over it effectively. The framework provides the tools needed to ingest data from diverse sources, structure it into intermediate representations (indexes), and query it through natural language interfaces.

The framework supports the full lifecycle of a Retrieval-Augmented Generation (RAG) application, from loading and parsing documents, to indexing and storing embeddings, to retrieving relevant context and synthesizing responses. Beyond basic RAG, LlamaIndex extends into multi-turn chat engines, autonomous agents with tool use, and event-driven workflows that orchestrate multi-step processes.

LlamaIndex offers a "5-line starter" experience where developers can load documents and begin querying with minimal code, while also exposing deep customization points for production-grade applications. The project maintains an active community across Discord, Twitter, and LinkedIn.

## Core Concepts

- **Data Connectors**: Ingest data from a wide variety of sources including APIs, PDFs, SQL databases, and other formats, normalizing them into document representations that the framework can process.
- **Indexes**: Intermediate data structures that organize ingested documents for efficient retrieval. Indexes store embeddings and metadata that enable semantic search over the underlying data.
- **Query Engines**: The primary interface for RAG-based question answering. A query engine takes a natural language query, retrieves relevant context from an index, and synthesizes a response using an LLM.
- **Chat Engines**: Extend query engines with conversational memory, enabling multi-turn interactions where the system maintains context across exchanges.
- **Agents**: LLM-powered workers equipped with tools that can reason, plan, and execute multi-step tasks autonomously. Agents use tool calling to interact with external systems and APIs.
- **Workflows**: Event-driven orchestration primitives that allow developers to compose multi-step, branching processes with explicit control flow over how data moves between stages.
- **Context Augmentation**: The foundational principle of making private or domain-specific data available to LLMs at inference time, bridging the gap between general-purpose models and specialized knowledge.

## Architecture

LlamaIndex is organized around a pipeline architecture that moves data through distinct stages:

1. **Loading**: Data connectors (also called readers) ingest raw data from sources such as files, APIs, databases, and web pages, producing `Document` objects.
2. **Indexing**: Documents are chunked, embedded, and organized into index structures. The most common index type is the `VectorStoreIndex`, which stores embeddings for semantic retrieval.
3. **Storing**: Indexes and their associated metadata can be persisted to vector stores, document stores, and index stores for reuse without re-processing.
4. **Querying**: Query engines and chat engines accept natural language input, retrieve relevant nodes from the index, and pass them as context to an LLM for response synthesis.
5. **Orchestration**: Agents and workflows coordinate multi-step processes, combining retrieval, tool use, and LLM reasoning into coherent task execution.

The framework follows a modular design where each component (LLM, embedding model, vector store, node parser) can be swapped independently, allowing developers to customize the stack for their specific requirements.

## Key Features and Functionality

- **Comprehensive Data Connectors**: Out-of-the-box support for loading data from APIs, PDFs, SQL databases, CSV files, web pages, and dozens of other formats through the LlamaHub ecosystem.
- **Flexible Indexing Strategies**: Multiple index types including vector indexes, keyword indexes, tree indexes, and knowledge graph indexes, each optimized for different retrieval patterns.
- **RAG Pipeline Primitives**: End-to-end support for building retrieval-augmented generation pipelines with customizable chunking, embedding, retrieval, and synthesis stages.
- **Multi-Turn Conversations**: Chat engines that maintain conversational state and context across multiple exchanges, enabling natural dialogue over data.
- **Autonomous Agents**: Agent abstractions that combine LLM reasoning with tool calling, allowing systems to plan and execute multi-step tasks.
- **Event-Driven Workflows**: A workflow system for building complex, multi-step processes with explicit event passing and control flow.
- **Multi-Modal Support**: Capabilities for working with both text and image data, enabling applications that reason over documents containing mixed media.
- **LlamaCloud Integration**: Managed services including LlamaParse for document parsing with Vision Language Models (VLMs), LlamaExtract for structured data extraction, and hosted indexing and retrieval pipelines.
- **Observability and Evaluation**: Built-in callbacks and integration points for tracing, logging, and evaluating RAG pipeline performance.
- **Broad LLM and Embedding Support**: Integrations with major LLM providers (OpenAI, Anthropic, Cohere, local models) and embedding services.

## Use Cases

- **Retrieval-Augmented Generation (RAG)**: Building question-answering systems that ground LLM responses in specific document collections, knowledge bases, or databases.
- **Conversational Chatbots**: Creating multi-turn chat interfaces over private data, such as customer support bots that reference internal documentation.
- **Document Understanding**: Parsing, indexing, and querying complex documents including PDFs, presentations, and structured reports.
- **Structured Data Extraction**: Extracting structured information from unstructured documents using LLM-powered parsing and schema-guided extraction.
- **Autonomous Agents**: Building agents that can plan, use tools, and execute multi-step tasks over data sources.
- **Multi-Modal Applications**: Processing and querying over documents that contain both text and images.
- **Fine-Tuning Data Preparation**: Using RAG pipelines to generate training data for fine-tuning domain-specific models.

## API Reference Summary

### Quick Start

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader

# Load documents from a directory
documents = SimpleDirectoryReader("data").load_data()

# Build an index over the documents
index = VectorStoreIndex.from_documents(documents)

# Create a query engine and ask questions
query_engine = index.as_query_engine()
response = query_engine.query("What is the main topic of these documents?")
print(response)
```

### Chat Engine

```python
chat_engine = index.as_chat_engine()
response = chat_engine.chat("Tell me about the key findings.")
follow_up = chat_engine.chat("Can you elaborate on the second point?")
```

### Custom LLM and Embedding Configuration

```python
from llama_index.core import Settings
from llama_index.llms.openai import OpenAI
from llama_index.embeddings.openai import OpenAIEmbedding

Settings.llm = OpenAI(model="gpt-4o", temperature=0.1)
Settings.embed_model = OpenAIEmbedding(model="text-embedding-3-small")
```

### Agent with Tools

```python
from llama_index.core.agent import ReActAgent
from llama_index.core.tools import QueryEngineTool

tool = QueryEngineTool.from_defaults(
    query_engine=query_engine,
    name="document_search",
    description="Search over the indexed documents"
)

agent = ReActAgent.from_tools([tool], verbose=True)
response = agent.chat("Find and summarize the key metrics.")
```

## Configuration and Customization

- **LLM Selection**: Configure the language model used for response synthesis through `Settings.llm`, supporting OpenAI, Anthropic, Cohere, local models via Ollama, and others.
- **Embedding Model**: Set the embedding model via `Settings.embed_model` to control how documents are vectorized for semantic search.
- **Chunk Size and Overlap**: Control document chunking behavior through `Settings.chunk_size` and `Settings.chunk_overlap` to tune retrieval granularity.
- **Node Parsers**: Customize how documents are split into nodes using sentence splitters, token splitters, or semantic chunkers.
- **Vector Stores**: Persist embeddings to external vector databases such as Chroma, Pinecone, Weaviate, Qdrant, or Milvus.
- **Callbacks and Observability**: Attach callback handlers for tracing, logging, and debugging pipeline execution.

## Integration Patterns

- **Vector Store Backends**: LlamaIndex integrates with major vector databases (Chroma, Pinecone, Weaviate, Qdrant, Milvus, pgvector) for persistent embedding storage and retrieval.
- **LLM Providers**: Supports OpenAI, Anthropic, Cohere, Google, Mistral, local models via Ollama and HuggingFace, and custom LLM implementations.
- **Data Sources via LlamaHub**: A community-driven hub of data connectors for ingesting data from Notion, Slack, Google Drive, databases, web scrapers, and hundreds of other sources.
- **Evaluation Frameworks**: Integrates with evaluation tools for measuring retrieval quality, answer relevance, and faithfulness.
- **LlamaCloud Services**: LlamaParse for parsing complex documents with vision language models, LlamaExtract for schema-guided structured extraction, and managed indexing and retrieval pipelines.

## Examples

### Basic RAG Pipeline

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader

documents = SimpleDirectoryReader("./data").load_data()
index = VectorStoreIndex.from_documents(documents)
query_engine = index.as_query_engine()
response = query_engine.query("Summarize the main findings.")
```

### Persistent Index with Chroma

```python
import chromadb
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, StorageContext
from llama_index.vector_stores.chroma import ChromaVectorStore

db = chromadb.PersistentClient(path="./chroma_db")
collection = db.get_or_create_collection("my_collection")
vector_store = ChromaVectorStore(chroma_collection=collection)
storage_context = StorageContext.from_defaults(vector_store=vector_store)

documents = SimpleDirectoryReader("./data").load_data()
index = VectorStoreIndex.from_documents(documents, storage_context=storage_context)
```

### Workflow-Based Pipeline

```python
from llama_index.core.workflow import Workflow, StartEvent, StopEvent, step

class RAGWorkflow(Workflow):
    @step
    async def retrieve(self, ev: StartEvent) -> StopEvent:
        query = ev.query
        nodes = self.index.as_retriever().retrieve(query)
        response = self.llm.complete(f"Context: {nodes}\nQuestion: {query}")
        return StopEvent(result=response)
```

## Limitations and Considerations

- **Learning Curve for Advanced Use**: While the 5-line starter is simple, production-grade configurations with custom retrievers, re-rankers, and multi-step agents require significant understanding of the framework internals.
- **Abstraction Overhead**: The high level of abstraction can make debugging retrieval quality issues more difficult, as the pipeline involves multiple layers between the query and the response.
- **Dependency Footprint**: The core package and its integrations pull in a substantial number of dependencies, which can increase deployment complexity.
- **Evolving API Surface**: As a rapidly developing framework, API changes between versions can require migration effort for existing applications.
- **Cloud Service Lock-In**: Some advanced features like LlamaParse and LlamaExtract are proprietary cloud services, creating a dependency on the LlamaCloud platform for those capabilities.

## Changelog Highlights

LlamaIndex is under active development with frequent releases. The project transitioned from a monolithic package to a modular architecture (`llama-index-core` plus integration packages) to improve maintainability and reduce dependency bloat. Key milestones include the introduction of the Workflows API for event-driven orchestration, expanded agent capabilities with tool calling, and the launch of LlamaCloud managed services.

## Citations

- [1] [LlamaIndex Documentation](https://developers.llamaindex.ai/python/framework/)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

LlamaIndex, RAG framework, data framework, context augmentation, VectorStoreIndex, SimpleDirectoryReader, query engine, chat engine, ReActAgent, LlamaHub, LlamaParse, LlamaExtract, LlamaCloud, data connectors, indexing, workflows, event-driven, tool calling, node parser, knowledge graph index, tree index, keyword index, multi-modal, RAG pipeline, document loading, semantic retrieval, 5-line starter

### Verb-Noun Tasks

- Load documents from a directory into an index
- Build a VectorStoreIndex from PDFs and APIs
- Ingest data via LlamaHub connectors
- Query an index with natural language
- Configure chunk size and overlap for retrieval
- Persist embeddings to a Chroma or Pinecone vector store
- Create a multi-turn chat engine over private data
- Wire a ReActAgent with QueryEngineTool
- Compose an event-driven RAG Workflow
- Parse complex PDFs with LlamaParse
- Extract structured data with LlamaExtract
- Swap the LLM via `Settings.llm`
- Trace pipeline execution with callbacks

### User Intent Phrases

- How do I build a RAG pipeline over my PDFs?
- I want a "5-line" starter that queries my documents
- How can I ingest data from Notion, Slack, and Google Drive?
- What is the easiest way to add chat memory over my data?
- How do I build an agent that can search my indexed documents?
- I need to parse complex PDF tables for RAG
- How do I switch from OpenAI to a local Ollama model in my RAG app?
- How do I persist my index to Pinecone or Qdrant?
- How do I orchestrate multi-step retrieval with branching logic?
- I want to extract structured fields from unstructured documents

### Problem Statements

- LLMs lack access to my private or domain-specific data
- Vanilla LLM responses are ungrounded and hallucinate
- I have hundreds of file formats and APIs to ingest before I can do RAG
- My retrieval pipeline needs custom chunking, embedding, and synthesis stages
- I need conversational memory on top of document retrieval
- Building agents that combine retrieval and tool use is hard from scratch
- Multi-step RAG processes need explicit control flow, not just a chain

### When to Pick This

- Pick this when you need the broadest data-connector ecosystem (LlamaHub) for ingesting heterogeneous sources
- Pick this when you want a "5-line starter" path from documents to queryable index
- Pick this over Haystack when you prefer flexible indexing strategies (vector, keyword, tree, knowledge graph) over typed pipeline graphs
- Pick this over GraphRAG when vector/keyword retrieval suffices and you do not need cross-document graph reasoning
- Pick this when LlamaCloud managed services (LlamaParse, LlamaExtract) fit your document-parsing needs
- Pick this when you want a single framework that spans RAG, chat engines, agents, and event-driven workflows

### Related Terms and Aliases

- GPT Index (legacy name)
- llama-index-core
- Retrieval-Augmented Generation (RAG)
- Document loader
- Indexing pipeline
- Vector store
- Tool-using agent
- ReAct agent
- LlamaHub
- LlamaParse
- LlamaCloud
