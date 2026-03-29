# Zep

> Context engineering platform with temporal knowledge graphs and agent memory for personalization

| Field | Value |
|-------|-------|
| Name | Zep |
| Group | Memory Systems |
| Type | API/SDK |
| Open Source | no |
| GitHub | N/A |
| Stars | N/A |
| Docs | [help.getzep.com](https://help.getzep.com/) |

## Overview

Zep is a context engineering platform that systematically assembles personalized context — user preferences, traits, and business data — for reliable agent applications. It combines agent memory, Graph Retrieval-Augmented Generation (RAG), and context assembly capabilities to deliver comprehensive personalized context that reduces hallucinations and improves accuracy [1].

The platform's distinguishing feature is its **temporal knowledge graph**, where nodes represent entities and edges represent facts and relationships that update dynamically. Facts carry temporal validity markers, tracking when information became valid and when it was invalidated by newer data [2].

Zep provides SDKs for Python, TypeScript, and Go, and integrates with agent frameworks including LangGraph, AutoGen, and CrewAI. The platform includes enterprise features such as HIPAA compliance, role-based access control (RBAC), and audit logging [1].

Zep's underlying graph technology is built on **Graphiti**, an open-source temporal knowledge graph framework (23,000+ GitHub stars) maintained by the Zep team [1].

## Core Concepts

### Temporal Knowledge Graphs

Zep's knowledge graph is its unified knowledge store. Nodes represent entities (people, places, concepts) and edges represent facts and relationships between them. A key temporal feature is **fact invalidation**: when new information supersedes prior knowledge, the system records when the old fact became invalid on that fact's edge, preserving the full temporal history [2].

### Graph Types

Zep supports two graph structures [2]:

- **Graph**: An arbitrary knowledge graph for storing current knowledge about objects or systems
- **User Graph**: A specialized graph for preserving personalized context specific to individual application users. All messages added to any thread of that user are ingested into the user's graph by default

### Users and Threads

**Users** represent individual application users, each with their own knowledge graph. Users should be created with at minimum a first name, ideally including last name and email for accurate entity identification in the graph [3].

**Threads** represent conversation threads belonging to a user. Messages added to any thread are automatically ingested into that user's knowledge graph. Threads are identified by unique IDs and associated with a user ID [3].

### Context Blocks

A **Context Block** is an optimized string containing a user summary and facts from the knowledge graph most relevant to the current thread. It includes dates when facts became valid and invalid. Context blocks are retrieved via `thread.get_user_context()` and combine semantic search, full-text search, and breadth-first search for comprehensive retrieval [2][3].

### Data Types

Zep ingests multiple data formats [2][3]:

- **Messages**: Chat history with user/assistant roles and timestamps
- **JSON**: Structured business data (transactions, events, user interactions)
- **Text**: Unstructured documents, emails, and support tickets

### Context Templates

Custom context templates allow developers to control the structure of retrieved context using template variables like `%{user_summary}`, `%{edges limit=10}`, and `%{entities limit=5}`. Templates are created once and referenced by ID when retrieving context [3].

## Architecture

Zep's architecture centers on a temporal knowledge graph that ingests data from multiple sources and assembles personalized context for agent consumption:

```
┌─────────────────────────────────────────────┐
│              Data Sources                    │
│  ┌──────────┐ ┌──────────┐ ┌─────────────┐  │
│  │  Chat    │ │ Business │ │  Documents  │  │
│  │ Messages │ │   Data   │ │  & Emails   │  │
│  │          │ │  (JSON)  │ │   (Text)    │  │
│  └────┬─────┘ └────┬─────┘ └──────┬──────┘  │
└───────┼─────────────┼──────────────┼─────────┘
        │             │              │
        v             v              v
┌─────────────────────────────────────────────┐
│         Temporal Knowledge Graph             │
│  ┌─────────────────────────────────────┐     │
│  │  Nodes (entities) ←→ Edges (facts)  │     │
│  │  + temporal validity markers         │     │
│  │  + fact invalidation tracking        │     │
│  └─────────────────────────────────────┘     │
│  ┌─────────┐  ┌────────────┐                 │
│  │  User   │  │  General   │                 │
│  │ Graphs  │  │  Graphs    │                 │
│  └─────────┘  └────────────┘                 │
└────────────────────┬────────────────────────┘
                     │
                     v
┌─────────────────────────────────────────────┐
│          Context Assembly Engine             │
│  Semantic + Full-text + BFS Search           │
│  User Summaries + Relevant Facts             │
│  Temporal Validity Markers                   │
│  Custom Context Templates                    │
└────────────────────┬────────────────────────┘
                     │
                     v
┌─────────────────────────────────────────────┐
│         Context Block (< 200ms)              │
│  → System Prompt or Tool Message             │
└─────────────────────────────────────────────┘
```

The platform retrieves context in sub-200ms latency, optimizing for **high recall over precision** — preferring inclusion of more results even if some are less relevant [3].

## Key Features

- **Temporal Knowledge Graphs**: Dynamic graph with nodes (entities) and edges (facts) that track temporal validity, including when facts become valid and invalid
- **Automatic Fact Extraction**: Ingested messages and data are automatically processed to extract entities, relationships, and facts into the knowledge graph
- **Fact Invalidation**: When new information supersedes prior knowledge, the old fact's invalidation time is preserved on the graph edge
- **Context Assembly**: Optimized context blocks combining user summaries and relevant facts with temporal validity markers, retrieved in sub-200ms
- **Custom Context Templates**: Configurable templates for controlling context structure using variables like `%{user_summary}`, `%{edges}`, `%{entities}`
- **Multi-Language SDKs**: Native SDKs for Python, TypeScript, and Go with consistent APIs
- **User Graphs**: Per-user knowledge graphs that automatically ingest all thread messages for personalization
- **Business Data Ingestion**: Support for JSON, text, and message data types representing transactions, events, documents, and emails
- **Batch Ingestion**: Bulk data loading for backfilling existing users and conversations
- **Graph RAG**: Graph-based retrieval augmented generation combining semantic search, full-text search, and breadth-first graph search
- **Agentic Tools**: Tool definitions enabling agents to directly query user knowledge graphs
- **Custom Entity and Edge Types**: Pydantic-like class definitions for specialized graph structures
- **HIPAA Compliance**: Enterprise-grade healthcare data compliance
- **RBAC and Audit Logging**: Role-based access control with comprehensive audit trails
- **Playground**: Web-based environment for testing graph queries and context retrieval

## Use Cases

- **Personalized AI Assistants**: Building agents that remember user preferences, traits, and history across conversations with temporal awareness
- **Customer Support**: Agents with access to customer interaction history, support tickets, and product knowledge via the knowledge graph
- **Healthcare Applications**: HIPAA-compliant memory for medical AI assistants tracking patient interactions and preferences
- **E-Commerce Personalization**: Ingesting purchase history, browsing behavior, and preferences as structured JSON data for recommendation agents
- **Music and Content Recommendation**: Tracking user listening/viewing behavior and extracting preference patterns through entity relationships
- **Multi-Agent Systems**: Shared knowledge graphs enabling context continuity across different specialized agents
- **Enterprise Knowledge Management**: Organizational graphs storing company-wide policies, procedures, and institutional knowledge

## API Reference

### User Management

```python
# Create user
user = client.user.add(
    user_id="internal_id",
    email="jane@example.com",
    first_name="Jane",
    last_name="Smith",
)

# Get user
user = client.user.get(user_id="internal_id")
```

### Thread Operations

```python
import uuid
from zep_cloud.types import Message
from datetime import datetime, timezone

# Create thread
thread_id = uuid.uuid4().hex
client.thread.create(thread_id=thread_id, user_id=user_id)

# Add messages (include name and RFC3339 timestamp)
messages = [
    Message(
        created_at=datetime.now(timezone.utc).isoformat(),
        name="Jane Smith",
        role="user",
        content="Who was Octavia Butler?",
    )
]
response = client.thread.add_messages(thread_id, messages=messages)

# Add assistant response
assistant_messages = [
    Message(
        created_at=datetime.now(timezone.utc).isoformat(),
        name="AI Assistant",
        role="assistant",
        content="Octavia Butler was an influential American science fiction writer...",
    )
]
client.thread.add_messages(thread_id, messages=assistant_messages)
```

### Context Retrieval

```python
# Default context block
user_context = client.thread.get_user_context(thread_id=thread_id)
context_block = user_context.context

# Custom template context
user_context = client.thread.get_user_context(
    thread_id=thread_id,
    template_id="customer-support",
)
```

### Graph Data Ingestion

```python
import json

# Add structured business data
event_data = {
    "user_id": "user123",
    "user_name": "Jane Smith",
    "event_type": "song_played",
    "song_title": "Bohemian Rhapsody",
    "artist": "Queen",
    "duration_seconds": 354,
}

client.graph.add(
    user_id="user123",
    type="json",
    data=json.dumps(event_data),
)
```

### Graph Search

```python
# Search the knowledge graph
results = client.graph.search(
    user_id="user123",
    query="music preferences",
)
```

### Context Templates

```python
# Create a custom template
client.context.create_context_template(
    template_id="customer-support",
    template="""# CUSTOMER PROFILE
%{user_summary}

# RECENT INTERACTIONS
%{edges limit=10}

# KEY ENTITIES
%{entities limit=5}""",
)
```

## Configuration

### API Key Setup

```bash
# Environment variable
export ZEP_API_KEY=your_api_key_here

# Or in .env file
ZEP_API_KEY=your_api_key_here
```

### Context Window Integration

Two recommended approaches for inserting Zep context into LLM calls [3]:

1. **System Prompt Injection**: Append the context block directly to the system prompt, refreshing dynamically on each turn
2. **Context Message Approach**: Insert the context block as a tool message after user messages, which enables prompt caching for improved efficiency

### Timestamp Format

All messages should use RFC3339 timestamp format for accurate temporal understanding in the knowledge graph [3].

### User Names in Messages

Including user names in messages is critical for accurate graph construction — the system uses names to identify and link entities in the knowledge graph [3].

### Backfilling Existing Data

For existing users and conversations, loop through data calling `user.add` and `thread.add_messages`, or use batch processing methods for large-scale ingestion [3].

## Integration Patterns

### LangGraph Integration

Zep integrates with LangGraph for building complex agent workflows with persistent memory and knowledge graph access [1].

### AutoGen Integration

Multi-agent AutoGen systems can leverage Zep for shared context and persistent memory across agent interactions [1].

### CrewAI Integration

CrewAI agent crews can use Zep as a memory backend for maintaining context across collaborative agent tasks [1].

### MCP Server

Zep provides a Model Context Protocol (MCP) server and `llms.txt` file for connecting AI coding assistants directly to Zep's documentation and capabilities [1].

### Agentic Tools

Zep provides tool definitions that enable agents to directly query user knowledge graphs during conversations, allowing agents to retrieve context autonomously [2].

## Examples

### Basic Conversation with Memory

```python
import os
import uuid
from zep_cloud.client import Zep
from zep_cloud.types import Message
from datetime import datetime, timezone

client = Zep(api_key=os.environ["ZEP_API_KEY"])

# Create user
user = client.user.add(
    user_id="jane_123",
    first_name="Jane",
    last_name="Smith",
    email="jane@example.com",
)

# Create thread
thread_id = uuid.uuid4().hex
client.thread.create(thread_id=thread_id, user_id="jane_123")

# Add conversation messages
messages = [
    Message(
        created_at=datetime.now(timezone.utc).isoformat(),
        name="Jane Smith",
        role="user",
        content="I'm vegetarian and I love Italian food.",
    )
]
client.thread.add_messages(thread_id, messages=messages)

# Retrieve personalized context for future interactions
user_context = client.thread.get_user_context(thread_id=thread_id)
print(user_context.context)
# Output includes: user summary, dietary preferences, temporal validity dates
```

### Business Data Ingestion

```python
import json

# Ingest purchase history
purchase = {
    "user_id": "jane_123",
    "user_name": "Jane Smith",
    "event_type": "purchase",
    "item": "Margherita Pizza Cookbook",
    "category": "books",
    "amount": 24.99,
}

client.graph.add(
    user_id="jane_123",
    type="json",
    data=json.dumps(purchase),
)

# The knowledge graph now links Jane to Italian cooking interests
# Future context blocks will include this preference
```

### Custom Context Template

```python
# Define a support-focused template
client.context.create_context_template(
    template_id="support-agent",
    template="""# USER PROFILE
%{user_summary}

# RELEVANT HISTORY
%{edges limit=15}

# KEY TOPICS
%{entities limit=8}""",
)

# Retrieve context using the template
ctx = client.thread.get_user_context(
    thread_id=thread_id,
    template_id="support-agent",
)

# Use in LLM system prompt
system_prompt = f"You are a support agent.\n\n{ctx.context}"
```

## Limitations

- **Closed source**: Zep is a managed service without a self-hosted open-source option (though Graphiti, the underlying graph framework, is open source)
- **API-dependent**: All operations require API connectivity to Zep's cloud infrastructure
- **High recall bias**: The platform optimizes for high recall over precision, which may return less relevant results alongside relevant ones [3]
- **Timestamp requirements**: RFC3339 format timestamps are required for accurate temporal tracking; missing or incorrect timestamps degrade temporal understanding [3]
- **Name dependency**: User names in messages are critical for accurate graph construction; anonymous messages may result in incomplete entity linking [3]
- **Asynchronous graph processing**: Data ingestion into the knowledge graph is asynchronous; recently added data may not be immediately available in context blocks
- **No offline/local deployment**: Unlike competitors, Zep does not offer a fully self-hosted deployment option

## Changelog

- **v3**: Current major version featuring temporal knowledge graphs, context assembly engine, custom context templates, and Graph RAG
- **Graphiti**: Open-source temporal knowledge graph framework extracted from Zep's core technology (23,000+ GitHub stars, 2,300+ forks)
- **MCP Server**: Added Model Context Protocol server for AI coding assistant integration
- **Mem0 Migration**: Published migration guide for users transitioning from Mem0 to Zep
- **Multi-language SDKs**: Python, TypeScript, and Go SDKs with consistent API surfaces

## Citations

- [1] Welcome to Zep - https://help.getzep.com/overview
- [2] Key Concepts - https://help.getzep.com/concepts
- [3] Quick Start Guide - https://help.getzep.com/quick-start-guide
