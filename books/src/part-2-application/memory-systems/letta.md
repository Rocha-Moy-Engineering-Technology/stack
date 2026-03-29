# Letta

> Platform for building stateful agents with persistent memory that learn and self-improve over time

| Field | Value |
|-------|-------|
| Name | Letta |
| Group | Memory Systems |
| Type | API/SDK |
| Open Source | yes |
| GitHub | [letta-ai/letta](https://github.com/letta-ai/letta) |
| Stars | 21392 |
| Docs | [docs.letta.com](https://docs.letta.com/) |

## Overview

Letta is a platform for building **stateful agents** that remember, learn, and improve over time. Unlike traditional Large Language Model (LLM) applications that treat each interaction as isolated, Letta agents maintain persistent knowledge across all interactions, forming living memories about themselves, the world they operate in, and the users they interact with.

The platform provides a complete infrastructure for managing agent state, including structured memory blocks, archival storage with semantic search, tool execution, multi-agent coordination through shared memory, and background memory processing via sleep-time agents. Letta is available as a hosted API service, a self-hosted Docker server, and as Letta Code — a memory-first coding agent for the terminal [1].

Letta originated from the MemGPT research project, which pioneered the concept of using operating system-inspired memory management techniques (virtual memory, paging) to enable LLMs to manage their own context windows effectively.

## Core Concepts

### Stateful Agents

A stateful agent in Letta manages growing knowledge while maintaining consistent behavior and incorporating new experiences. The system stores all state — memories, user messages, reasoning traces, and tool calls — in a database, ensuring information survives context window limitations. Critical memories are injected into the LLM's active context, while the agent can self-modify its memories through dedicated memory tools [2].

### Memory Blocks

Memory blocks are structured sections of the agent's context window that persist across all interactions. They are prepended to prompts in XML-like formatting, making them immediately visible to the LLM without requiring retrieval. Each block has four components [3]:

- **Label**: Unique identifier (e.g., `persona`, `human`, `organization`)
- **Description**: Explains the block's purpose — the primary signal agents use to decide how to read and write to a block
- **Value**: The actual content stored in the block
- **Limit**: Character size restriction

Agents autonomously organize information within blocks based on their labels and descriptions. Blocks can be configured as read-only to prevent agent modification while maintaining visibility. The recommended maximum size is under 50,000 characters per block [5].

### Archival Memory

Archival memory is a semantically searchable database where agents store facts, knowledge, and information for long-term retrieval. Unlike memory blocks (which are always in-context), archival memory entries exist outside the context window and require active querying through tools [4]:

- `archival_memory_insert`: Store new information with optional tags for categorization
- `archival_memory_search`: Query memories semantically (e.g., searching "artificial memories" returns results about "implanted memories")

Archival memory is agent-immutable by design — agents cannot easily modify or delete entries, though developers retain full control via SDK. It scales to practically unlimited capacity and supports tag-based organization [4].

### Context Hierarchy

Letta provides four tiers of context abstractions, with placement strategy based on data scale [5]:

- **Memory Blocks**: In-context, editable, for critical information under ~50k characters. Tools: `memory_rethink`, `memory_replace`, `memory_insert`
- **Files**: Partially in-context (openable/closable), read-only, up to 5MB per file. Tools: `open`, `close`, `semantic_search`, `grep`
- **Archival Memory**: Out-of-context, read-write, 300-token passages. Tools: `archival_memory_insert`, `archival_memory_search`
- **External RAG**: Out-of-context, unlimited scale, accessed via custom tools or Model Context Protocol (MCP)

### Shared Memory

Shared memory blocks enable multiple agents to access and modify the same memory simultaneously. When one agent updates a shared block, all connected agents see the change immediately, enabling real-time coordination without explicit agent-to-agent messaging [6].

Concurrency safety varies by operation:
- `memory_insert` (appending): concurrent-safe
- `memory_replace` (targeted edits): mostly safe
- `memory_rethink` (complete rewrites): unsafe (last-writer-wins)

Best practice is to designate one agent as the "owner" for major edits, with other agents using append-only operations.

### Runs, Steps, and Conversations

Agent invocations are structured as **runs**, where a single run may contain sequential **steps** performing multiple LLM inference passes. **Conversations** enable concurrent messaging threads using the same agent with different users [2].

### AgentFile (.af)

AgentFile is an open standard file format for serializing stateful agents into a single portable file. It packages model configuration, message history, system prompt, memory blocks, tool rules, environment variables, and tool definitions. Agents can be exported and imported via the SDK, REST API, or the Agent Development Environment (ADE) [8].

## Architecture

Letta's architecture centers on persistent state management for agents:

```
┌──────────────────────────────────────────┐
│              Letta Platform              │
│                                          │
│  ┌─────────────────────────────────┐     │
│  │           Agent                 │     │
│  │  ┌───────────┐ ┌────────────┐  │     │
│  │  │  System   │ │  Memory    │  │     │
│  │  │  Prompt   │ │  Blocks    │  │     │
│  │  └───────────┘ └────────────┘  │     │
│  │  ┌───────────┐ ┌────────────┐  │     │
│  │  │ Messages  │ │   Tools    │  │     │
│  │  └───────────┘ └────────────┘  │     │
│  └─────────────────────────────────┘     │
│                                          │
│  ┌──────────┐ ┌──────────┐ ┌─────────┐  │
│  │ Archival │ │  Files   │ │ External│  │
│  │ Memory   │ │          │ │   RAG   │  │
│  │ (Vector) │ │ (Search) │ │  (MCP)  │  │
│  └──────────┘ └──────────┘ └─────────┘  │
│                                          │
│  ┌──────────────────────────────────┐    │
│  │     PostgreSQL + pgvector        │    │
│  └──────────────────────────────────┘    │
└──────────────────────────────────────────┘
```

All agent state is stored in PostgreSQL with the pgvector extension for semantic search. The system manages the context window by placing memory blocks directly in-context and providing tools for agents to access out-of-context data stores. This approach allows agents to effectively manage unbounded knowledge using a finite context window [2][7].

### Sleep-Time Agents

Sleep-time agents are an experimental multi-agent architecture where background agents asynchronously process and consolidate memories. When enabled, the system creates a primary (interactive) agent and a sleep-time agent that shares memory blocks. The sleep-time agent triggers every N steps (default: 5) to reflect on conversation history and derive important insights, writing "learned context" back to shared memory blocks [10].

## Key Features

- **Persistent Memory**: All agent state (memories, messages, reasoning, tool calls) stored in a database and survives across sessions
- **Self-Modifying Memory**: Agents autonomously read, write, and reorganize their own memory blocks using built-in memory tools
- **Shared Memory Blocks**: Multiple agents access and modify the same memory in real-time for coordination without explicit messaging
- **Archival Memory**: Semantically searchable long-term storage with tag-based organization and unlimited capacity
- **Context Hierarchy**: Four-tier system (blocks, files, archival, external RAG) for managing data at different scales
- **Sleep-Time Compute**: Background agents asynchronously consolidate and refine memories between interactions
- **Multi-Provider Models**: Support for OpenAI, Anthropic, Google AI, Azure, AWS Bedrock, OpenRouter, and Ollama with hot-swappable models
- **Tool Ecosystem**: Built-in tools (web search, code interpreter, fetch), custom server tools, MCP tools, and client-side tools
- **AgentFile (.af)**: Open standard format for serializing and sharing complete stateful agents
- **Agent Development Environment (ADE)**: Web-based interface for building, testing, and managing agents
- **Bring Your Own Keys (BYOK)**: Direct billing through your own API provider accounts
- **Role-Based Access Control (RBAC)**: Granular permissions for production deployments

## Use Cases

- **Personal AI Assistants**: Agents that build deep user profiles over thousands of interactions, remembering preferences, relationships, and conversation history
- **Customer Support**: Agents that accumulate institutional knowledge, track customer histories, and improve responses based on past resolutions
- **Coding Agents**: Letta Code provides a memory-first coding agent that remembers your codebase, patterns, and preferences across sessions
- **Multi-Agent Coordination**: Supervisor/worker patterns where supervisors write tasks to shared memory and workers read and update status
- **Research Assistants**: Agents maintaining academic literature repositories with semantic cross-referencing in archival memory
- **Social Media Agents**: Agents tracking tens of thousands of user interactions with persistent memory

## API Reference

### Client Initialization

```python
from letta_client import Letta

# Hosted API
client = Letta(api_key="your-api-key")

# Self-hosted Docker
client = Letta(base_url="http://localhost:8283")
```

### Agent Operations

```python
# Create agent with memory blocks
agent = client.agents.create(
    model="openai/gpt-4.1",
    memory_blocks=[
        {"label": "human", "value": "Name: Alice"},
        {"label": "persona", "value": "You are a helpful assistant."},
    ],
)

# Send message
response = client.agents.messages.create(
    agent_id=agent.id,
    messages=[{"role": "user", "content": "Hello!"}],
)

# Process response
for message in response.messages:
    if hasattr(message, "content"):
        print(message.content)
```

### Memory Block Management

```python
# Create standalone block
block = client.blocks.create(
    label="organization",
    description="Shared company information",
    value="Company policies...",
)

# Attach to agent
client.agents.blocks.attach(agent_id=agent.id, block_id=block.id)

# Retrieve by label
block = client.agents.blocks.retrieve(agent_id=agent.id, block_label="persona")

# Update block value
client.blocks.update(block_id=block.id, value="Updated content")
```

### Archival Memory

```python
# Insert passage
client.agents.passages.create(
    agent_id=agent.id,
    text="Important fact to remember",
)

# Search semantically
results = client.agents.passages.list(
    agent_id=agent.id,
    query_text="related concept",
)
```

### Tool Creation

```python
def roll_dice() -> str:
    """Simulate 20-sided die roll (d20).

    Returns random integer between 1-20.

    Returns:
        str: Die roll outcome.
    """
    import random
    return f"You rolled a {random.randint(1, 20)}"

tool = client.tools.create_from_function(func=roll_dice)

# Attach tool to agent
agent = client.agents.create(
    model="openai/gpt-4.1",
    tools=[tool.name],
    memory_blocks=[{"label": "persona", "value": "You love games."}],
)
```

### Export/Import Agents

```python
# Export
schema = client.agents.export_file(agent_id=agent.id)

# Import
imported = client.agents.import_file(file=open("agent.af", "rb"))
```

## Configuration

### Model Selection

Models are specified using the `provider/model-name` format:

```python
agent = client.agents.create(
    model="anthropic/claude-sonnet-4-5-20250929",
    memory_blocks=[...],
)
```

Supported providers: `openai/`, `anthropic/`, `google_ai/`, `azure/`, `bedrock/`, `openrouter/`, `ollama/` [11].

### Docker Environment Variables

| Variable | Purpose |
|----------|---------|
| `OPENAI_API_KEY` | OpenAI model access |
| `ANTHROPIC_API_KEY` | Anthropic model access |
| `GOOGLE_AI_API_KEY` | Google AI model access |
| `OLLAMA_BASE_URL` | Local Ollama endpoint |
| `LETTA_PG_URI` | Custom PostgreSQL connection (requires pgvector) |
| `E2B_API_KEY` | Sandbox for custom tool execution |
| `EXA_API_KEY` | Web search and webpage fetch enhancement |
| `SECURE` / `LETTA_SERVER_PASSWORD` | Authentication for production |

### Embedding Models

When using Docker, embedding models must be specified explicitly:

```python
agent = client.agents.create(
    model="openai/gpt-4o-mini",
    embedding="openai/text-embedding-3-small",
)
```

The hosted API handles embedding configuration automatically [7].

### Sleep-Time Configuration

```python
# Enable sleep-time on agent creation
agent = client.agents.create(
    model="anthropic/claude-sonnet-4-5-20250929",
    memory_blocks=[...],
    enable_sleeptime=True,
)

# Adjust frequency (triggers every N steps)
from letta_client import SleeptimeManagerUpdate
group = client.groups.update(
    group_id=group_id,
    manager_config=SleeptimeManagerUpdate(sleeptime_agent_frequency=5),
)
```

Recommended frequency: 5-10 steps. Lower values increase token usage with diminishing returns [10].

## Integration Patterns

### Multi-Agent via Shared Blocks

```python
# Create shared block
shared = client.blocks.create(
    label="task_board",
    description="Shared task tracking between agents",
    value="",
)

# Create supervisor with write access
supervisor = client.agents.create(
    model="openai/gpt-4.1",
    memory_blocks=[{"label": "persona", "value": "You are a supervisor."}],
    block_ids=[shared.id],
)

# Create worker with same shared block
worker = client.agents.create(
    model="openai/gpt-4.1",
    memory_blocks=[{"label": "persona", "value": "You are a worker."}],
    block_ids=[shared.id],
)
```

### MCP Tool Integration

Letta agents can connect to MCP servers for external tool access. MCP tools function as schema-only definitions on the Letta side, with execution delegated to the MCP server [12].

### Server Tools with Injected Client

Custom server tools automatically receive environment variables (`LETTA_AGENT_ID`, `LETTA_PROJECT_ID`, `LETTA_API_KEY`) and a pre-initialized `client` object, enabling tools to access the Letta API for dynamic memory management and sub-agent creation [13].

```python
def get_my_memory() -> dict:
    """Retrieve current agent's memory blocks."""
    import os
    agent_id = os.environ.get('LETTA_AGENT_ID')
    agent = client.agents.retrieve(agent_id=agent_id)
    return {block.label: block.value for block in agent.memory.blocks}
```

## Examples

### Basic Chat Agent with Persistent Memory

```python
from letta_client import Letta

client = Letta(api_key="your-key")

agent = client.agents.create(
    model="openai/gpt-4.1",
    memory_blocks=[
        {"label": "human", "value": "The user hasn't introduced themselves yet."},
        {"label": "persona", "value": "You are a friendly assistant who remembers everything."},
    ],
)

# First conversation
response = client.agents.messages.create(
    agent_id=agent.id,
    messages=[{"role": "user", "content": "Hi! I'm Alice, I work at Acme Corp."}],
)

# Later conversation — agent remembers Alice
response = client.agents.messages.create(
    agent_id=agent.id,
    messages=[{"role": "user", "content": "What do you remember about me?"}],
)
```

### Agent with Archival Knowledge Base

```python
agent = client.agents.create(
    model="openai/gpt-4.1",
    memory_blocks=[
        {"label": "persona", "value": "You are a research assistant."},
    ],
    tools=["archival_memory_insert", "archival_memory_search"],
)

# Seed archival memory with documents
for doc in documents:
    client.agents.passages.create(agent_id=agent.id, text=doc)

# Agent can now search its knowledge base during conversations
response = client.agents.messages.create(
    agent_id=agent.id,
    messages=[{"role": "user", "content": "What do we know about quantum computing?"}],
)
```

### Sleep-Time Agent for Background Learning

```python
agent = client.agents.create(
    model="anthropic/claude-sonnet-4-5-20250929",
    embedding="openai/text-embedding-3-small",
    memory_blocks=[
        {"label": "human", "value": ""},
        {"label": "persona", "value": "You are a helpful assistant."},
    ],
    enable_sleeptime=True,
)

# As conversations happen, the sleep-time agent
# asynchronously processes and consolidates memories
# into refined memory blocks
```

## Limitations

- **Sleep-time agents are experimental**: The feature may be unstable and is subject to change [10]
- **Docker embedding requirement**: Self-hosted deployments must explicitly specify embedding models, unlike the hosted API [7]
- **Context window constraints**: Memory blocks consume context window space; very large blocks (>50k characters) may degrade performance [5]
- **Shared memory concurrency**: `memory_rethink` operations on shared blocks are unsafe under concurrent access (last-writer-wins) [6]
- **AgentFile secrets**: Exported `.af` files null out secrets for security; re-configuration is needed after import [8]
- **Tool sandboxing**: Docker deployments require E2B API key for custom tool sandboxing; TypeScript server tools also require E2B on Docker [13]
- **HTTPS requirement**: The ADE requires HTTPS connections except for localhost access [7]
- **Code interpreter statefulness**: Each code execution runs in a fresh environment without state retention between calls [14]

## Changelog

Letta evolved from the **MemGPT** research project (2023), which introduced OS-inspired virtual memory management for LLMs. The project rebranded to Letta and expanded into a full platform offering:

- **MemGPT era**: Research prototype demonstrating self-editing memory and context management for LLMs
- **Letta Platform**: Production-ready hosted API with managed infrastructure
- **Letta Code**: Memory-first coding agent for terminal use (Node.js based)
- **Letta Code SDK**: TypeScript SDK for building apps on top of stateful computer use agents
- **AgentFile (.af)**: Open standard for portable agent serialization
- **Sleep-time compute**: Experimental background memory consolidation (research paper: arxiv.org/abs/2504.13171)

## Citations

- [1] Letta Platform Landing Page - https://docs.letta.com/
- [2] Stateful Agents - https://docs.letta.com/guides/core-concepts/stateful-agents/
- [3] Memory Blocks - https://docs.letta.com/guides/core-concepts/memory/memory-blocks/
- [4] Archival Memory - https://docs.letta.com/guides/core-concepts/memory/archival-memory/
- [5] Context Hierarchy - https://docs.letta.com/guides/core-concepts/memory/context-hierarchy/
- [6] Shared Memory - https://docs.letta.com/guides/core-concepts/memory/shared-memory/
- [7] Docker Server Setup - https://docs.letta.com/guides/docker/
- [8] AgentFile (.af) - https://docs.letta.com/guides/core-concepts/agent-file/
- [9] Quickstart (API) - https://docs.letta.com/guides/build-with-letta/quickstart/
- [10] Sleep-Time Agents - https://docs.letta.com/guides/agents/architectures/sleeptime/
- [11] Models - https://docs.letta.com/guides/build-with-letta/models/
- [12] MCP Tools - https://docs.letta.com/guides/core-concepts/tools/mcp-tools/
- [13] Server Tools - https://docs.letta.com/guides/core-concepts/tools/server-tools/
- [14] Built-in Tools - https://docs.letta.com/guides/core-concepts/tools/builtin-tools/
