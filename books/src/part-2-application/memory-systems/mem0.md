# Mem0

> Universal memory layer for AI agents with persistent personalized memory capabilities

| Field | Value |
|-------|-------|
| Name | Mem0 |
| Group | Memory Systems |
| Type | API/SDK |
| Open Source | yes |
| GitHub | [mem0ai/mem0](https://github.com/mem0ai/mem0) |
| Stars | 55709 |
| Docs | [docs.mem0.ai](https://docs.mem0.ai/introduction) |

## Overview

Mem0 is a universal, self-improving memory layer for Large Language Model (LLM) applications. It provides persistent memory infrastructure that enables AI agents to retain and learn from interactions over time, moving beyond stateless request-response patterns. The platform automatically extracts key facts from conversations, resolves conflicts with existing memories, and stores them for semantic retrieval [1].

Mem0 is available in three deployment models [1]:

- **Mem0 Platform**: Fully managed service with production-scale infrastructure, SOC 2 Type II compliance, and built-in graph services, vector stores, and rerankers
- **Mem0 Open Source**: Self-hosted option providing full control over data, deployment, and customization with no vendor lock-in
- **OpenMemory**: Workspace-focused product for teams collaborating across agents and projects

The platform integrates with 20+ AI frameworks including LangChain, CrewAI, Vercel AI SDK, AutoGen, LlamaIndex, and LangGraph [8].

## Core Concepts

### Memory Types

Mem0 organizes memory into four hierarchical layers [5]:

- **Conversation Memory**: In-flight messages within a single turn, including tool outputs and intermediate calculations. Expires after the current turn completes
- **Session Memory**: Short-lived facts lasting minutes to hours, ideal for multi-step workflows like onboarding or debugging. Scoped by `session_id`
- **User Memory**: Long-lived knowledge tied to a person, account, or workspace. Persists weeks to indefinitely across sessions. Scoped by `user_id`
- **Organizational Memory**: Shared context globally configured for multiple agents or teams, containing FAQs, product catalogs, and policies

The system captures details at the conversation layer and **promotes** relevant information upward based on identifiers. During retrieval, the pipeline ranks results from user memories first, followed by session notes, then raw history [5].

### Memory Processing Pipeline

When memories are added, Mem0 follows a three-stage pipeline [6]:

1. **Information Extraction**: An LLM identifies key facts, decisions, and preferences from the conversation
2. **Conflict Resolution**: The system checks existing memories for duplicates or contradictions, ensuring the latest truth wins
3. **Storage**: Memories are stored in vector storage (plus optional graph storage) for retrieval

### Graph Memory

Graph Memory augments the standard vector search by automatically establishing connections between entities in stored data. When enabled, the system extracts entities (people, locations, jobs) and determines their relationships [4]:

- Vector search returns top semantic matches with optional reranking
- Graph relations are returned alongside vector results to provide additional context
- Entity relationships include source, target, relationship type, and confidence scores

Graph memory is enabled per-call with `enable_graph=True` or globally at the project level [4].

### Search Pipeline

Memory retrieval follows four stages [7]:

1. **Query Processing**: Natural-language questions are cleaned and enriched for embedding search
2. **Vector Search**: Embeddings locate closest memories via cosine similarity
3. **Filtering and Reranking**: Logical filters (AND/OR, comparison operators) narrow candidates; optional rerankers refine ordering
4. **Results Delivery**: Formatted memories with metadata, timestamps, and relevance scores are returned

## Architecture

Mem0's architecture consists of a memory processing engine layered over configurable storage backends:

```
┌───────────────────────────────────────────┐
│             Application Layer             │
│   (Python SDK / JS SDK / REST API)        │
├───────────────────────────────────────────┤
│           Memory Processing Engine        │
│  ┌─────────┐ ┌───────────┐ ┌──────────┐  │
│  │ Extract  │ │ Conflict  │ │  Store   │  │
│  │  Facts   │→│ Resolve   │→│  Memory  │  │
│  └─────────┘ └───────────┘ └──────────┘  │
├───────────────────────────────────────────┤
│            Storage Backends               │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │  Vector  │ │  Graph   │ │ History  │  │
│  │  Store   │ │  Store   │ │   DB     │  │
│  │ (Qdrant) │ │ (Neo4j)  │ │(SQLite)  │  │
│  └──────────┘ └──────────┘ └──────────┘  │
├───────────────────────────────────────────┤
│           LLM + Embedder + Reranker       │
└───────────────────────────────────────────┘
```

The Platform version manages all infrastructure components (vector stores, graph services, rerankers) as a hosted service. The Open Source version requires users to configure and run each component [2][3].

## Key Features

- **Automatic Memory Extraction**: LLM-powered extraction of key facts, preferences, and decisions from conversations with conflict resolution against existing memories
- **Semantic Search**: Natural language queries with cosine similarity matching, optional reranking, and configurable similarity thresholds
- **Graph Memory**: Relationship-aware recall that extracts entities and their connections, returning graph relations alongside vector search results
- **Four Memory Layers**: Conversation, session, user, and organizational memory with automatic promotion and hierarchical retrieval
- **Metadata Filtering**: JSON-based logical filters (AND/OR, comparison operators) for date ranges, categories, and custom metadata
- **Multi-Framework Integration**: Native support for LangChain, CrewAI, Vercel AI SDK, AutoGen, LlamaIndex, LangGraph, and 15+ other frameworks
- **MCP Support**: Model Context Protocol (MCP) server for universal AI client connectivity
- **Async-by-Default**: Asynchronous client support since v1.0.0 for high-throughput applications
- **Reranking**: Configurable reranker support (Cohere, Zero Entropy) for improved retrieval precision
- **Multimodal Support**: Memory operations supporting multiple content modalities
- **Custom Categories**: User-defined categorization for organizing and filtering memories
- **Webhook Integrations**: Event-driven notifications for memory operations
- **Enterprise Controls**: SOC 2 Type II compliance, GDPR adherence, audit logs, and workspace governance (Platform)

## Use Cases

- **Personal AI Assistants**: Building agents that remember user preferences, dietary restrictions, travel plans, and conversation history across sessions
- **Customer Support**: Agents accumulating institutional knowledge and tracking customer histories to avoid repetitive questions
- **Multi-Agent Coordination**: Shared organizational memory enabling teams of agents to access consistent context
- **Onboarding Workflows**: Session memory tracking multi-step processes with bounded timeframes
- **Recommendation Systems**: Storing and retrieving user preferences for personalized suggestions (movies, restaurants, products)
- **Voice Agents**: Integration with LiveKit, ElevenLabs, and Pipecat for conversational AI with persistent memory
- **RAG Enhancement**: Augmenting Retrieval-Augmented Generation (RAG) pipelines with persistent user context

## API Reference

### Add Memory

```python
from mem0 import MemoryClient

client = MemoryClient(api_key="your-api-key")

messages = [
    {"role": "user", "content": "I'm planning a trip to Tokyo next month."},
    {"role": "assistant", "content": "Great! I'll remember that for future suggestions."}
]

# Platform
result = client.add(messages=messages, user_id="alice")

# With metadata and graph
result = client.add(
    messages=messages,
    user_id="alice",
    metadata={"category": "travel"},
    enable_graph=True,
)
```

### Search Memory

```python
# Basic search
results = client.search("What do you know about me?", filters={"user_id": "alice"})

# With advanced filters (Platform)
results = client.search(
    "hotel preferences",
    filters={
        "AND": [
            {"user_id": "alice"},
            {"categories": {"contains": "travel"}},
        ]
    },
)

# Open Source
from mem0 import Memory
m = Memory()
results = m.search("hotel preferences", user_id="alice")
```

### Update Memory

```python
client.update(memory_id="mem-id-123", data="Updated preference: boutique hotels in Shibuya")
```

### Delete Memory

```python
# Delete specific memory
client.delete(memory_id="mem-id-123")

# Delete all memories for a user
client.delete_all(filters={"user_id": "alice"})
```

### Get All Memories

```python
# Platform
memories = client.get_all(filters={"AND": [{"user_id": "alice"}]})

# Open Source
memories = m.get_all(user_id="alice")
```

### REST API

```bash
# Add memory
curl -X POST https://api.mem0.ai/v1/memories/add \
  -H "Authorization: Token your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "I love sushi"}], "user_id": "alice"}'

# Search memory
curl -X POST https://api.mem0.ai/v1/memories/search \
  -H "Authorization: Token your-api-key" \
  -H "Content-Type: application/json" \
  -d '{"query": "food preferences", "filters": {"user_id": "alice"}}'
```

## Configuration

### Open Source Configuration

Mem0 OSS uses a dictionary-based configuration system with four configurable components: vector stores, LLMs, embedders, and rerankers [9]:

```python
from mem0 import Memory

config = {
    "llm": {
        "provider": "openai",
        "config": {
            "model": "gpt-4.1-mini",
            "temperature": 0.2,
        }
    },
    "embedder": {
        "provider": "ollama",
        "config": {
            "model": "nomic-embed-text",
        }
    },
    "vector_store": {
        "provider": "qdrant",
        "config": {
            "collection_name": "my_memories",
            "host": "localhost",
            "port": 6333,
        }
    },
}

m = Memory.from_config(config)
# Or from file: m = Memory.from_config_file("config.yaml")
```

### Supported Providers

| Component | Providers |
|-----------|-----------|
| LLM | OpenAI, Azure OpenAI, Anthropic, Ollama, local models |
| Embedder | OpenAI, Vertex AI, Ollama, Cohere |
| Vector Store | Qdrant, PostgreSQL (pgvector), managed alternatives |
| Graph Store | Neo4j, Memgraph |
| Reranker | Cohere, Zero Entropy |

### Configuration Best Practices

- Keep extraction temperatures at 0.2 or below for deterministic memory processing
- Limit reranker `top_k` to 10-20 results
- Name vector collections explicitly in production for tenant isolation
- Store API credentials in environment variables [9]

### Platform Configuration

The Platform manages infrastructure automatically. Configuration is done at the project level:

```python
# Enable graph memory for all operations
client.project.update(enable_graph=True)
```

## Integration Patterns

### LangChain Integration

```python
from langchain.memory import Mem0Memory

memory = Mem0Memory(api_key="your-key", user_id="alice")
# Use as LangChain memory backend
```

### CrewAI Integration

Mem0 serves as a shared memory backend for CrewAI agent crews, enabling persistent context across collaborative agent interactions [8].

### MCP Integration

Mem0 provides an MCP server for universal AI client connectivity, enabling any MCP-compatible client to manage memory autonomously [2].

### Vercel AI SDK

Integration with the Vercel AI SDK enables memory-powered applications in Next.js and other JavaScript frameworks [8].

## Examples

### Personalized Assistant with Memory

```python
from mem0 import MemoryClient

client = MemoryClient(api_key="your-api-key")

# Store user preferences from conversation
messages = [
    {"role": "user", "content": "I'm vegetarian and allergic to nuts."},
    {"role": "assistant", "content": "I've noted your dietary preferences."},
    {"role": "user", "content": "I prefer boutique hotels over large chains."},
    {"role": "assistant", "content": "Got it! Boutique hotels for your travels."},
]
client.add(messages=messages, user_id="alice")

# Later, retrieve relevant context
results = client.search("restaurant suggestions", filters={"user_id": "alice"})
# Returns: memories about vegetarian preference and nut allergy
```

### Graph-Enhanced Memory

```python
messages = [
    {"role": "user", "content": "My name is Joseph. I'm from Seattle and work as a software engineer."},
]
client.add(messages, user_id="joseph", enable_graph=True)

# Search returns both vector matches and entity relationships
results = client.search("what is my name?", user_id="joseph", enable_graph=True)
# Results include relations: joseph -> lives_in -> Seattle, joseph -> works_as -> software_engineer
```

### Multi-Session Context

```python
# Session 1: Trip planning
client.add(
    [{"role": "user", "content": "I want to visit Tokyo in March."}],
    user_id="alex",
    session_id="trip-planning-2025",
)

# Session 2: Different context, same user memory
results = client.search(
    "Any travel plans?",
    user_id="alex",
)
# Returns Tokyo trip memory from previous session
```

## Limitations

- **OpenAI dependency**: Default open source configuration requires an OpenAI API key; alternative providers require explicit configuration [3]
- **Async processing for graph**: Adding memories with graph enabled is asynchronous; memories may not be immediately available for retrieval [4]
- **Inference mode mixing**: Using both `infer=True` and `infer=False` for identical content creates duplicate memories [6]
- **Date filtering**: Date range filters are available only on the Platform, not in the open source version [7]
- **Security consideration**: The retrieval-by-design architecture means stored content is accessible by any query scoped to the same identifiers; avoid storing unencrypted secrets or personally identifiable information [5]
- **Graph memory overhead**: Graph processing introduces additional latency, though generally acceptable for most use cases [4]
- **Reranking disabled by default**: Must be explicitly configured for improved retrieval precision [3]

## Changelog

- **v1.0.0**: Major release shipping rerankers, async-by-default behavior, Azure OpenAI support, and breaking API changes [2]
- **Pre-v1.0**: Initial releases establishing core memory operations, vector search, and Python/JavaScript SDK support
- **Graph Memory**: Added relationship-aware recall with Neo4j and Memgraph support
- **OpenMemory**: Introduced workspace-based memory for multi-agent team collaboration
- **MCP Support**: Added Model Context Protocol server for universal AI client integration
- **Multimodal**: Added support for multimodal memory content

## Citations

- [1] Welcome to Mem0 - https://docs.mem0.ai/introduction
- [2] Platform Overview - https://docs.mem0.ai/platform/overview
- [3] Open Source Overview - https://docs.mem0.ai/open-source/overview
- [4] Graph Memory - https://docs.mem0.ai/platform/features/graph-memory
- [5] Memory Types - https://docs.mem0.ai/core-concepts/memory-types
- [6] Add Memory - https://docs.mem0.ai/core-concepts/memory-operations/add
- [7] Search Memory - https://docs.mem0.ai/core-concepts/memory-operations/search
- [8] Integrations - https://docs.mem0.ai/integrations
- [9] Configuration - https://docs.mem0.ai/open-source/configuration

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- Mem0
- universal memory layer
- self-improving memory
- automatic memory extraction
- conflict resolution
- four memory layers
- conversation memory
- session memory
- user memory
- organizational memory
- graph memory
- semantic search
- vector + graph
- Qdrant backend
- Neo4j graph store
- pgvector option
- reranker
- enable_graph
- MCP server
- OpenMemory
- multi-framework integrations
- user_id scoped memory
- SOC 2 Type II

### Verb-Noun Tasks

- Add a memory from a conversation with `client.add(messages, user_id=...)`
- Extract facts automatically from chat history via LLM
- Resolve conflicts when new facts contradict existing memories
- Search memories semantically by natural-language query
- Enable graph memory to capture entity relationships
- Filter memories with JSON logical filters (AND/OR, comparison operators)
- Promote conversation-layer facts to session, user, or org memory
- Add a reranker (Cohere, Zero Entropy) for retrieval precision
- Self-host Mem0 with Qdrant + Neo4j + OpenAI embeddings
- Integrate Mem0 with LangChain, CrewAI, AutoGen, LangGraph, LlamaIndex
- Expose Mem0 to any AI client via its MCP server
- Build voice agents with persistent memory (LiveKit, ElevenLabs, Pipecat)

### User Intent Phrases

- How do I give my LLM app persistent memory across sessions?
- How can my agent remember user preferences automatically?
- What handles conflict when a user updates their preference?
- How do I add a memory layer between my chat app and the LLM?
- How can I get entity relationships, not just vector similarity?
- How do I scope memories per user, session, and organization?
- How do I let multiple agents share organizational context?
- How do I self-host a memory backend with Qdrant?
- How do I integrate persistent memory into a LangGraph or CrewAI agent?
- What is the simplest API for adding memory to an LLM application?

### Problem Statements

- LLM apps are stateless and forget context across sessions
- Manual fact extraction and conflict resolution is brittle
- Vector search alone misses entity relationships
- Default OSS config requires OpenAI; alternatives need explicit setup
- Graph-enabled adds are asynchronous; recent memories may not be retrievable immediately
- Mixing `infer=True` and `infer=False` for identical content creates duplicates
- Date filtering only available on Platform, not OSS

### When to Pick This

- Pick Mem0 when you want a turnkey memory layer with automatic extraction, conflict resolution, and broad framework integrations
- Pick Mem0 for simple user preference tracking — fastest to integrate among dedicated memory platforms
- Pick Mem0 when you want both vector similarity AND optional graph relationships in one API
- Pick Mem0 OSS when you need self-hosted control with no vendor lock-in
- Pick Letta instead when agents should self-modify their own memory blocks (stateful agent OS, four-tier context hierarchy with sleep-time compute)
- Pick Zep instead when facts change over time and you need temporal knowledge graphs with fact invalidation (support agents, CRM)

### Related Terms and Aliases

- universal AI memory
- persistent agent memory
- LLM long-term memory
- conversation memory layer
- OpenMemory
- memory-as-a-service
- vector + graph memory
- agent recall layer
- memory client SDK
