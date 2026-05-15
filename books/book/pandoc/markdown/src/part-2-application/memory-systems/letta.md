[Header 1 ("letta", [], []) [Str "Letta"], BlockQuote [Para [Str "Platform for building stateful agents with persistent memory that learn and self-improve over time"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Letta"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Memory Systems"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "letta-ai/letta"] ("https://github.com/letta-ai/letta", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "22716"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "docs.letta.com"] ("https://docs.letta.com/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Letta is a platform for building ", Strong [Str "stateful agents"], Str " that remember, learn, and improve over time. Unlike traditional Large Language Model (LLM) applications that treat each interaction as isolated, Letta agents maintain persistent knowledge across all interactions, forming living memories about themselves, the world they operate in, and the users they interact with."], Para [Str "The platform provides a complete infrastructure for managing agent state, including structured memory blocks, archival storage with semantic search, tool execution, multi-agent coordination through shared memory, and background memory processing via sleep-time agents. Letta is available as a hosted API service, a self-hosted Docker server, and as Letta Code — a memory-first coding agent for the terminal ", Str "[", Str "1", Str "]", Str "."], Para [Str "Letta originated from the MemGPT research project, which pioneered the concept of using operating system-inspired memory management techniques (virtual memory, paging) to enable LLMs to manage their own context windows effectively."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("stateful-agents", ["unnumbered", "unlisted"], []) [Str "Stateful Agents"], Para [Str "A stateful agent in Letta manages growing knowledge while maintaining consistent behavior and incorporating new experiences. The system stores all state — memories, user messages, reasoning traces, and tool calls — in a database, ensuring information survives context window limitations. Critical memories are injected into the LLM's active context, while the agent can self-modify its memories through dedicated memory tools ", Str "[", Str "2", Str "]", Str "."], Header 3 ("memory-blocks", ["unnumbered", "unlisted"], []) [Str "Memory Blocks"], Para [Str "Memory blocks are structured sections of the agent's context window that persist across all interactions. They are prepended to prompts in XML-like formatting, making them immediately visible to the LLM without requiring retrieval. Each block has four components ", Str "[", Str "3", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Label"], Str ": Unique identifier (e.g., ", Code ("", [], []) "persona", Str ", ", Code ("", [], []) "human", Str ", ", Code ("", [], []) "organization", Str ")"]], [Plain [Strong [Str "Description"], Str ": Explains the block's purpose — the primary signal agents use to decide how to read and write to a block"]], [Plain [Strong [Str "Value"], Str ": The actual content stored in the block"]], [Plain [Strong [Str "Limit"], Str ": Character size restriction"]]], Para [Str "Agents autonomously organize information within blocks based on their labels and descriptions. Blocks can be configured as read-only to prevent agent modification while maintaining visibility. The recommended maximum size is under 50,000 characters per block ", Str "[", Str "5", Str "]", Str "."], Header 3 ("archival-memory", ["unnumbered", "unlisted"], []) [Str "Archival Memory"], Para [Str "Archival memory is a semantically searchable database where agents store facts, knowledge, and information for long-term retrieval. Unlike memory blocks (which are always in-context), archival memory entries exist outside the context window and require active querying through tools ", Str "[", Str "4", Str "]", Str ":"], BulletList [[Plain [Code ("", [], []) "archival_memory_insert", Str ": Store new information with optional tags for categorization"]], [Plain [Code ("", [], []) "archival_memory_search", Str ": Query memories semantically (e.g., searching \"artificial memories\" returns results about \"implanted memories\")"]]], Para [Str "Archival memory is agent-immutable by design — agents cannot easily modify or delete entries, though developers retain full control via SDK. It scales to practically unlimited capacity and supports tag-based organization ", Str "[", Str "4", Str "]", Str "."], Header 3 ("context-hierarchy", ["unnumbered", "unlisted"], []) [Str "Context Hierarchy"], Para [Str "Letta provides four tiers of context abstractions, with placement strategy based on data scale ", Str "[", Str "5", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Memory Blocks"], Str ": In-context, editable, for critical information under ", Str "~", Str "50k characters. Tools: ", Code ("", [], []) "memory_rethink", Str ", ", Code ("", [], []) "memory_replace", Str ", ", Code ("", [], []) "memory_insert"]], [Plain [Strong [Str "Files"], Str ": Partially in-context (openable/closable), read-only, up to 5MB per file. Tools: ", Code ("", [], []) "open", Str ", ", Code ("", [], []) "close", Str ", ", Code ("", [], []) "semantic_search", Str ", ", Code ("", [], []) "grep"]], [Plain [Strong [Str "Archival Memory"], Str ": Out-of-context, read-write, 300-token passages. Tools: ", Code ("", [], []) "archival_memory_insert", Str ", ", Code ("", [], []) "archival_memory_search"]], [Plain [Strong [Str "External RAG"], Str ": Out-of-context, unlimited scale, accessed via custom tools or Model Context Protocol (MCP)"]]], Header 3 ("shared-memory", ["unnumbered", "unlisted"], []) [Str "Shared Memory"], Para [Str "Shared memory blocks enable multiple agents to access and modify the same memory simultaneously. When one agent updates a shared block, all connected agents see the change immediately, enabling real-time coordination without explicit agent-to-agent messaging ", Str "[", Str "6", Str "]", Str "."], Para [Str "Concurrency safety varies by operation:"], BulletList [[Plain [Code ("", [], []) "memory_insert", Str " (appending): concurrent-safe"]], [Plain [Code ("", [], []) "memory_replace", Str " (targeted edits): mostly safe"]], [Plain [Code ("", [], []) "memory_rethink", Str " (complete rewrites): unsafe (last-writer-wins)"]]], Para [Str "Best practice is to designate one agent as the \"owner\" for major edits, with other agents using append-only operations."], Header 3 ("runs-steps-and-conversations", ["unnumbered", "unlisted"], []) [Str "Runs, Steps, and Conversations"], Para [Str "Agent invocations are structured as ", Strong [Str "runs"], Str ", where a single run may contain sequential ", Strong [Str "steps"], Str " performing multiple LLM inference passes. ", Strong [Str "Conversations"], Str " enable concurrent messaging threads using the same agent with different users ", Str "[", Str "2", Str "]", Str "."], Header 3 ("agentfile-af", ["unnumbered", "unlisted"], []) [Str "AgentFile (.af)"], Para [Str "AgentFile is an open standard file format for serializing stateful agents into a single portable file. It packages model configuration, message history, system prompt, memory blocks, tool rules, environment variables, and tool definitions. Agents can be exported and imported via the SDK, REST API, or the Agent Development Environment (ADE) ", Str "[", Str "8", Str "]", Str "."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Letta's architecture centers on persistent state management for agents:"], CodeBlock ("", [""], []) "┌──────────────────────────────────────────┐
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
", Para [Str "All agent state is stored in PostgreSQL with the pgvector extension for semantic search. The system manages the context window by placing memory blocks directly in-context and providing tools for agents to access out-of-context data stores. This approach allows agents to effectively manage unbounded knowledge using a finite context window ", Str "[", Str "2", Str "]", Str "[", Str "7", Str "]", Str "."], Header 3 ("sleep-time-agents", ["unnumbered", "unlisted"], []) [Str "Sleep-Time Agents"], Para [Str "Sleep-time agents are an experimental multi-agent architecture where background agents asynchronously process and consolidate memories. When enabled, the system creates a primary (interactive) agent and a sleep-time agent that shares memory blocks. The sleep-time agent triggers every N steps (default: 5) to reflect on conversation history and derive important insights, writing \"learned context\" back to shared memory blocks ", Str "[", Str "10", Str "]", Str "."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Persistent Memory"], Str ": All agent state (memories, messages, reasoning, tool calls) stored in a database and survives across sessions"]], [Plain [Strong [Str "Self-Modifying Memory"], Str ": Agents autonomously read, write, and reorganize their own memory blocks using built-in memory tools"]], [Plain [Strong [Str "Shared Memory Blocks"], Str ": Multiple agents access and modify the same memory in real-time for coordination without explicit messaging"]], [Plain [Strong [Str "Archival Memory"], Str ": Semantically searchable long-term storage with tag-based organization and unlimited capacity"]], [Plain [Strong [Str "Context Hierarchy"], Str ": Four-tier system (blocks, files, archival, external RAG) for managing data at different scales"]], [Plain [Strong [Str "Sleep-Time Compute"], Str ": Background agents asynchronously consolidate and refine memories between interactions"]], [Plain [Strong [Str "Multi-Provider Models"], Str ": Support for OpenAI, Anthropic, Google AI, Azure, AWS Bedrock, OpenRouter, and Ollama with hot-swappable models"]], [Plain [Strong [Str "Tool Ecosystem"], Str ": Built-in tools (web search, code interpreter, fetch), custom server tools, MCP tools, and client-side tools"]], [Plain [Strong [Str "AgentFile (.af)"], Str ": Open standard format for serializing and sharing complete stateful agents"]], [Plain [Strong [Str "Agent Development Environment (ADE)"], Str ": Web-based interface for building, testing, and managing agents"]], [Plain [Strong [Str "Bring Your Own Keys (BYOK)"], Str ": Direct billing through your own API provider accounts"]], [Plain [Strong [Str "Role-Based Access Control (RBAC)"], Str ": Granular permissions for production deployments"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Personal AI Assistants"], Str ": Agents that build deep user profiles over thousands of interactions, remembering preferences, relationships, and conversation history"]], [Plain [Strong [Str "Customer Support"], Str ": Agents that accumulate institutional knowledge, track customer histories, and improve responses based on past resolutions"]], [Plain [Strong [Str "Coding Agents"], Str ": Letta Code provides a memory-first coding agent that remembers your codebase, patterns, and preferences across sessions"]], [Plain [Strong [Str "Multi-Agent Coordination"], Str ": Supervisor/worker patterns where supervisors write tasks to shared memory and workers read and update status"]], [Plain [Strong [Str "Research Assistants"], Str ": Agents maintaining academic literature repositories with semantic cross-referencing in archival memory"]], [Plain [Strong [Str "Social Media Agents"], Str ": Agents tracking tens of thousands of user interactions with persistent memory"]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("client-initialization", ["unnumbered", "unlisted"], []) [Str "Client Initialization"], CodeBlock ("", ["python"], []) "from letta_client import Letta

# Hosted API
client = Letta(api_key=\"your-api-key\")

# Self-hosted Docker
client = Letta(base_url=\"http://localhost:8283\")
", Header 3 ("agent-operations", ["unnumbered", "unlisted"], []) [Str "Agent Operations"], CodeBlock ("", ["python"], []) "# Create agent with memory blocks
agent = client.agents.create(
    model=\"openai/gpt-4.1\",
    memory_blocks=[
        {\"label\": \"human\", \"value\": \"Name: Alice\"},
        {\"label\": \"persona\", \"value\": \"You are a helpful assistant.\"},
    ],
)

# Send message
response = client.agents.messages.create(
    agent_id=agent.id,
    messages=[{\"role\": \"user\", \"content\": \"Hello!\"}],
)

# Process response
for message in response.messages:
    if hasattr(message, \"content\"):
        print(message.content)
", Header 3 ("memory-block-management", ["unnumbered", "unlisted"], []) [Str "Memory Block Management"], CodeBlock ("", ["python"], []) "# Create standalone block
block = client.blocks.create(
    label=\"organization\",
    description=\"Shared company information\",
    value=\"Company policies...\",
)

# Attach to agent
client.agents.blocks.attach(agent_id=agent.id, block_id=block.id)

# Retrieve by label
block = client.agents.blocks.retrieve(agent_id=agent.id, block_label=\"persona\")

# Update block value
client.blocks.update(block_id=block.id, value=\"Updated content\")
", Header 3 ("archival-memory-1", ["unnumbered", "unlisted"], []) [Str "Archival Memory"], CodeBlock ("", ["python"], []) "# Insert passage
client.agents.passages.create(
    agent_id=agent.id,
    text=\"Important fact to remember\",
)

# Search semantically
results = client.agents.passages.list(
    agent_id=agent.id,
    query_text=\"related concept\",
)
", Header 3 ("tool-creation", ["unnumbered", "unlisted"], []) [Str "Tool Creation"], CodeBlock ("", ["python"], []) "def roll_dice() -> str:
    \"\"\"Simulate 20-sided die roll (d20).

    Returns random integer between 1-20.

    Returns:
        str: Die roll outcome.
    \"\"\"
    import random
    return f\"You rolled a {random.randint(1, 20)}\"

tool = client.tools.create_from_function(func=roll_dice)

# Attach tool to agent
agent = client.agents.create(
    model=\"openai/gpt-4.1\",
    tools=[tool.name],
    memory_blocks=[{\"label\": \"persona\", \"value\": \"You love games.\"}],
)
", Header 3 ("exportimport-agents", ["unnumbered", "unlisted"], []) [Str "Export/Import Agents"], CodeBlock ("", ["python"], []) "# Export
schema = client.agents.export_file(agent_id=agent.id)

# Import
imported = client.agents.import_file(file=open(\"agent.af\", \"rb\"))
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("model-selection", ["unnumbered", "unlisted"], []) [Str "Model Selection"], Para [Str "Models are specified using the ", Code ("", [], []) "provider/model-name", Str " format:"], CodeBlock ("", ["python"], []) "agent = client.agents.create(
    model=\"anthropic/claude-sonnet-4-5-20250929\",
    memory_blocks=[...],
)
", Para [Str "Supported providers: ", Code ("", [], []) "openai/", Str ", ", Code ("", [], []) "anthropic/", Str ", ", Code ("", [], []) "google_ai/", Str ", ", Code ("", [], []) "azure/", Str ", ", Code ("", [], []) "bedrock/", Str ", ", Code ("", [], []) "openrouter/", Str ", ", Code ("", [], []) "ollama/", Str " ", Str "[", Str "11", Str "]", Str "."], Header 3 ("docker-environment-variables", ["unnumbered", "unlisted"], []) [Str "Docker Environment Variables"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Variable"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Purpose"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "OPENAI_API_KEY"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "OpenAI model access"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "ANTHROPIC_API_KEY"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Anthropic model access"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "GOOGLE_AI_API_KEY"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Google AI model access"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "OLLAMA_BASE_URL"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Local Ollama endpoint"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "LETTA_PG_URI"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Custom PostgreSQL connection (requires pgvector)"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "E2B_API_KEY"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Sandbox for custom tool execution"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "EXA_API_KEY"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Web search and webpage fetch enhancement"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Code ("", [], []) "SECURE", Str " / ", Code ("", [], []) "LETTA_SERVER_PASSWORD"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Authentication for production"]]]])] (TableFoot ("", [], []) []), Header 3 ("embedding-models", ["unnumbered", "unlisted"], []) [Str "Embedding Models"], Para [Str "When using Docker, embedding models must be specified explicitly:"], CodeBlock ("", ["python"], []) "agent = client.agents.create(
    model=\"openai/gpt-4o-mini\",
    embedding=\"openai/text-embedding-3-small\",
)
", Para [Str "The hosted API handles embedding configuration automatically ", Str "[", Str "7", Str "]", Str "."], Header 3 ("sleep-time-configuration", ["unnumbered", "unlisted"], []) [Str "Sleep-Time Configuration"], CodeBlock ("", ["python"], []) "# Enable sleep-time on agent creation
agent = client.agents.create(
    model=\"anthropic/claude-sonnet-4-5-20250929\",
    memory_blocks=[...],
    enable_sleeptime=True,
)

# Adjust frequency (triggers every N steps)
from letta_client import SleeptimeManagerUpdate
group = client.groups.update(
    group_id=group_id,
    manager_config=SleeptimeManagerUpdate(sleeptime_agent_frequency=5),
)
", Para [Str "Recommended frequency: 5-10 steps. Lower values increase token usage with diminishing returns ", Str "[", Str "10", Str "]", Str "."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("multi-agent-via-shared-blocks", ["unnumbered", "unlisted"], []) [Str "Multi-Agent via Shared Blocks"], CodeBlock ("", ["python"], []) "# Create shared block
shared = client.blocks.create(
    label=\"task_board\",
    description=\"Shared task tracking between agents\",
    value=\"\",
)

# Create supervisor with write access
supervisor = client.agents.create(
    model=\"openai/gpt-4.1\",
    memory_blocks=[{\"label\": \"persona\", \"value\": \"You are a supervisor.\"}],
    block_ids=[shared.id],
)

# Create worker with same shared block
worker = client.agents.create(
    model=\"openai/gpt-4.1\",
    memory_blocks=[{\"label\": \"persona\", \"value\": \"You are a worker.\"}],
    block_ids=[shared.id],
)
", Header 3 ("mcp-tool-integration", ["unnumbered", "unlisted"], []) [Str "MCP Tool Integration"], Para [Str "Letta agents can connect to MCP servers for external tool access. MCP tools function as schema-only definitions on the Letta side, with execution delegated to the MCP server ", Str "[", Str "12", Str "]", Str "."], Header 3 ("server-tools-with-injected-client", ["unnumbered", "unlisted"], []) [Str "Server Tools with Injected Client"], Para [Str "Custom server tools automatically receive environment variables (", Code ("", [], []) "LETTA_AGENT_ID", Str ", ", Code ("", [], []) "LETTA_PROJECT_ID", Str ", ", Code ("", [], []) "LETTA_API_KEY", Str ") and a pre-initialized ", Code ("", [], []) "client", Str " object, enabling tools to access the Letta API for dynamic memory management and sub-agent creation ", Str "[", Str "13", Str "]", Str "."], CodeBlock ("", ["python"], []) "def get_my_memory() -> dict:
    \"\"\"Retrieve current agent's memory blocks.\"\"\"
    import os
    agent_id = os.environ.get('LETTA_AGENT_ID')
    agent = client.agents.retrieve(agent_id=agent_id)
    return {block.label: block.value for block in agent.memory.blocks}
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-chat-agent-with-persistent-memory", ["unnumbered", "unlisted"], []) [Str "Basic Chat Agent with Persistent Memory"], CodeBlock ("", ["python"], []) "from letta_client import Letta

client = Letta(api_key=\"your-key\")

agent = client.agents.create(
    model=\"openai/gpt-4.1\",
    memory_blocks=[
        {\"label\": \"human\", \"value\": \"The user hasn't introduced themselves yet.\"},
        {\"label\": \"persona\", \"value\": \"You are a friendly assistant who remembers everything.\"},
    ],
)

# First conversation
response = client.agents.messages.create(
    agent_id=agent.id,
    messages=[{\"role\": \"user\", \"content\": \"Hi! I'm Alice, I work at Acme Corp.\"}],
)

# Later conversation — agent remembers Alice
response = client.agents.messages.create(
    agent_id=agent.id,
    messages=[{\"role\": \"user\", \"content\": \"What do you remember about me?\"}],
)
", Header 3 ("agent-with-archival-knowledge-base", ["unnumbered", "unlisted"], []) [Str "Agent with Archival Knowledge Base"], CodeBlock ("", ["python"], []) "agent = client.agents.create(
    model=\"openai/gpt-4.1\",
    memory_blocks=[
        {\"label\": \"persona\", \"value\": \"You are a research assistant.\"},
    ],
    tools=[\"archival_memory_insert\", \"archival_memory_search\"],
)

# Seed archival memory with documents
for doc in documents:
    client.agents.passages.create(agent_id=agent.id, text=doc)

# Agent can now search its knowledge base during conversations
response = client.agents.messages.create(
    agent_id=agent.id,
    messages=[{\"role\": \"user\", \"content\": \"What do we know about quantum computing?\"}],
)
", Header 3 ("sleep-time-agent-for-background-learning", ["unnumbered", "unlisted"], []) [Str "Sleep-Time Agent for Background Learning"], CodeBlock ("", ["python"], []) "agent = client.agents.create(
    model=\"anthropic/claude-sonnet-4-5-20250929\",
    embedding=\"openai/text-embedding-3-small\",
    memory_blocks=[
        {\"label\": \"human\", \"value\": \"\"},
        {\"label\": \"persona\", \"value\": \"You are a helpful assistant.\"},
    ],
    enable_sleeptime=True,
)

# As conversations happen, the sleep-time agent
# asynchronously processes and consolidates memories
# into refined memory blocks
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Sleep-time agents are experimental"], Str ": The feature may be unstable and is subject to change ", Str "[", Str "10", Str "]"]], [Plain [Strong [Str "Docker embedding requirement"], Str ": Self-hosted deployments must explicitly specify embedding models, unlike the hosted API ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Context window constraints"], Str ": Memory blocks consume context window space; very large blocks (>50k characters) may degrade performance ", Str "[", Str "5", Str "]"]], [Plain [Strong [Str "Shared memory concurrency"], Str ": ", Code ("", [], []) "memory_rethink", Str " operations on shared blocks are unsafe under concurrent access (last-writer-wins) ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "AgentFile secrets"], Str ": Exported ", Code ("", [], []) ".af", Str " files null out secrets for security; re-configuration is needed after import ", Str "[", Str "8", Str "]"]], [Plain [Strong [Str "Tool sandboxing"], Str ": Docker deployments require E2B API key for custom tool sandboxing; TypeScript server tools also require E2B on Docker ", Str "[", Str "13", Str "]"]], [Plain [Strong [Str "HTTPS requirement"], Str ": The ADE requires HTTPS connections except for localhost access ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Code interpreter statefulness"], Str ": Each code execution runs in a fresh environment without state retention between calls ", Str "[", Str "14", Str "]"]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], Para [Str "Letta evolved from the ", Strong [Str "MemGPT"], Str " research project (2023), which introduced OS-inspired virtual memory management for LLMs. The project rebranded to Letta and expanded into a full platform offering:"], BulletList [[Plain [Strong [Str "MemGPT era"], Str ": Research prototype demonstrating self-editing memory and context management for LLMs"]], [Plain [Strong [Str "Letta Platform"], Str ": Production-ready hosted API with managed infrastructure"]], [Plain [Strong [Str "Letta Code"], Str ": Memory-first coding agent for terminal use (Node.js based)"]], [Plain [Strong [Str "Letta Code SDK"], Str ": TypeScript SDK for building apps on top of stateful computer use agents"]], [Plain [Strong [Str "AgentFile (.af)"], Str ": Open standard for portable agent serialization"]], [Plain [Strong [Str "Sleep-time compute"], Str ": Experimental background memory consolidation (research paper: arxiv.org/abs/2504.13171)"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Letta Platform Landing Page - https://docs.letta.com/"]], [Plain [Str "[", Str "2", Str "]", Str " Stateful Agents - https://docs.letta.com/guides/core-concepts/stateful-agents/"]], [Plain [Str "[", Str "3", Str "]", Str " Memory Blocks - https://docs.letta.com/guides/core-concepts/memory/memory-blocks/"]], [Plain [Str "[", Str "4", Str "]", Str " Archival Memory - https://docs.letta.com/guides/core-concepts/memory/archival-memory/"]], [Plain [Str "[", Str "5", Str "]", Str " Context Hierarchy - https://docs.letta.com/guides/core-concepts/memory/context-hierarchy/"]], [Plain [Str "[", Str "6", Str "]", Str " Shared Memory - https://docs.letta.com/guides/core-concepts/memory/shared-memory/"]], [Plain [Str "[", Str "7", Str "]", Str " Docker Server Setup - https://docs.letta.com/guides/docker/"]], [Plain [Str "[", Str "8", Str "]", Str " AgentFile (.af) - https://docs.letta.com/guides/core-concepts/agent-file/"]], [Plain [Str "[", Str "9", Str "]", Str " Quickstart (API) - https://docs.letta.com/guides/build-with-letta/quickstart/"]], [Plain [Str "[", Str "10", Str "]", Str " Sleep-Time Agents - https://docs.letta.com/guides/agents/architectures/sleeptime/"]], [Plain [Str "[", Str "11", Str "]", Str " Models - https://docs.letta.com/guides/build-with-letta/models/"]], [Plain [Str "[", Str "12", Str "]", Str " MCP Tools - https://docs.letta.com/guides/core-concepts/tools/mcp-tools/"]], [Plain [Str "[", Str "13", Str "]", Str " Server Tools - https://docs.letta.com/guides/core-concepts/tools/server-tools/"]], [Plain [Str "[", Str "14", Str "]", Str " Built-in Tools - https://docs.letta.com/guides/core-concepts/tools/builtin-tools/"]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "Letta"]], [Plain [Str "MemGPT"]], [Plain [Str "stateful agents"]], [Plain [Str "self-modifying memory"]], [Plain [Str "memory blocks"]], [Plain [Str "archival memory"]], [Plain [Str "context hierarchy"]], [Plain [Str "four-tier context"]], [Plain [Str "sleep-time agents"]], [Plain [Str "sleep-time compute"]], [Plain [Str "shared memory blocks"]], [Plain [Str "agent operating system"]], [Plain [Str "pgvector storage"]], [Plain [Str "PostgreSQL backend"]], [Plain [Str "AgentFile (.af)"]], [Plain [Str "Agent Development Environment"]], [Plain [Str "ADE"]], [Plain [Str "Letta Code"]], [Plain [Str "archival_memory_search"]], [Plain [Str "archival_memory_insert"]], [Plain [Str "memory_rethink"]], [Plain [Str "memory_replace"]], [Plain [Str "memory_insert"]], [Plain [Str "runs and steps"]], [Plain [Str "BYOK"]], [Plain [Str "RBAC"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Create a stateful agent with persistent memory blocks"]], [Plain [Str "Let an agent self-modify its memory via ", Code ("", [], []) "memory_rethink", Str " / ", Code ("", [], []) "memory_replace", Str " / ", Code ("", [], []) "memory_insert"]], [Plain [Str "Insert facts into archival memory with optional tags"]], [Plain [Str "Search archival memory semantically with ", Code ("", [], []) "archival_memory_search"]], [Plain [Str "Share memory blocks across multiple agents for real-time coordination"]], [Plain [Str "Enable sleep-time agents for background memory consolidation"]], [Plain [Str "Export and import full agent state as a ", Code ("", [], []) ".af", Str " AgentFile"]], [Plain [Str "Hot-swap LLM providers per-agent (OpenAI, Anthropic, Bedrock, Ollama)"]], [Plain [Str "Build supervisor/worker patterns via shared task-board blocks"]], [Plain [Str "Run Letta self-hosted on Docker with pgvector"]], [Plain [Str "Open files (up to 5MB) inside the agent's context with ", Code ("", [], []) "open", Str "/", Code ("", [], []) "close", Str "/", Code ("", [], []) "grep", Str "/", Code ("", [], []) "semantic_search"]], [Plain [Str "Manage agents through the web-based Agent Development Environment"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "How do I build agents that remember everything across sessions?"]], [Plain [Str "How can an agent edit its own memory autonomously?"]], [Plain [Str "I want an agent that learns and self-improves over time"]], [Plain [Str "How do I share memory between multiple agents?"]], [Plain [Str "How can background agents consolidate memories asynchronously?"]], [Plain [Str "What is the modern descendant of MemGPT?"]], [Plain [Str "How do I package an entire agent (memory, tools, prompts) into one file?"]], [Plain [Str "How do I build a coding agent that remembers my codebase across sessions?"]], [Plain [Str "How do I run a stateful agent platform self-hosted?"]], [Plain [Str "How does Letta differ from a simple vector memory layer?"]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "LLM apps treat each interaction as isolated; agents don't grow"]], [Plain [Str "Vector-only memory layers cannot represent self-modifying state and reasoning traces"]], [Plain [Str "Context window limits force ad-hoc retrieval architectures"]], [Plain [Str "Sleep-time agents are experimental and may be unstable"]], [Plain [Str "Memory blocks consume context window space; very large blocks degrade performance"]], [Plain [Str "Concurrent ", Code ("", [], []) "memory_rethink", Str " on shared blocks is unsafe (last-writer-wins)"]], [Plain [Str "Docker deployments must explicitly specify embedding models (hosted handles automatically)"]], [Plain [Str "ADE requires HTTPS except localhost"]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick Letta when agents need to autonomously read, write, and reorganize their own memory (true stateful agent OS, not just a memory layer)"]], [Plain [Str "Pick Letta when you want a four-tier context hierarchy (blocks, files, archival, external RAG) with sleep-time background consolidation"]], [Plain [Str "Pick Letta when multi-agent coordination should happen via shared memory blocks rather than message passing"]], [Plain [Str "Pick Letta when you want to serialize and ship a complete agent (memory + tools + prompts) as a portable ", Code ("", [], []) ".af", Str " file"]], [Plain [Str "Pick Mem0 instead when you need a simple memory layer with broad framework integrations and turnkey extraction"]], [Plain [Str "Pick Zep instead when facts change over time and you need temporal knowledge graphs with fact invalidation"]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "MemGPT descendant"]], [Plain [Str "stateful agent platform"]], [Plain [Str "agent OS"]], [Plain [Str "self-editing memory"]], [Plain [Str "LLM virtual memory"]], [Plain [Str "agent paging"]], [Plain [Str "memory-first coding agent"]], [Plain [Str "Letta Code"]], [Plain [Str "AgentFile standard"]], [Plain [Str "sleep-time compute"]], [Plain [Str "arxiv 2504.13171"]]]]