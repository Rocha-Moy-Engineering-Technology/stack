[Header 1 ("mem0", [], []) [Str "Mem0"], BlockQuote [Para [Str "Universal memory layer for AI agents with persistent personalized memory capabilities"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Mem0"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Memory Systems"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "mem0ai/mem0"] ("https://github.com/mem0ai/mem0", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "48695"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "docs.mem0.ai"] ("https://docs.mem0.ai/introduction", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Mem0 is a universal, self-improving memory layer for Large Language Model (LLM) applications. It provides persistent memory infrastructure that enables AI agents to retain and learn from interactions over time, moving beyond stateless request-response patterns. The platform automatically extracts key facts from conversations, resolves conflicts with existing memories, and stores them for semantic retrieval ", Str "[", Str "1", Str "]", Str "."], Para [Str "Mem0 is available in three deployment models ", Str "[", Str "1", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Mem0 Platform"], Str ": Fully managed service with production-scale infrastructure, SOC 2 Type II compliance, and built-in graph services, vector stores, and rerankers"]], [Plain [Strong [Str "Mem0 Open Source"], Str ": Self-hosted option providing full control over data, deployment, and customization with no vendor lock-in"]], [Plain [Strong [Str "OpenMemory"], Str ": Workspace-focused product for teams collaborating across agents and projects"]]], Para [Str "The platform integrates with 20+ AI frameworks including LangChain, CrewAI, Vercel AI SDK, AutoGen, LlamaIndex, and LangGraph ", Str "[", Str "8", Str "]", Str "."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("memory-types", ["unnumbered", "unlisted"], []) [Str "Memory Types"], Para [Str "Mem0 organizes memory into four hierarchical layers ", Str "[", Str "5", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Conversation Memory"], Str ": In-flight messages within a single turn, including tool outputs and intermediate calculations. Expires after the current turn completes"]], [Plain [Strong [Str "Session Memory"], Str ": Short-lived facts lasting minutes to hours, ideal for multi-step workflows like onboarding or debugging. Scoped by ", Code ("", [], []) "session_id"]], [Plain [Strong [Str "User Memory"], Str ": Long-lived knowledge tied to a person, account, or workspace. Persists weeks to indefinitely across sessions. Scoped by ", Code ("", [], []) "user_id"]], [Plain [Strong [Str "Organizational Memory"], Str ": Shared context globally configured for multiple agents or teams, containing FAQs, product catalogs, and policies"]]], Para [Str "The system captures details at the conversation layer and ", Strong [Str "promotes"], Str " relevant information upward based on identifiers. During retrieval, the pipeline ranks results from user memories first, followed by session notes, then raw history ", Str "[", Str "5", Str "]", Str "."], Header 3 ("memory-processing-pipeline", ["unnumbered", "unlisted"], []) [Str "Memory Processing Pipeline"], Para [Str "When memories are added, Mem0 follows a three-stage pipeline ", Str "[", Str "6", Str "]", Str ":"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Information Extraction"], Str ": An LLM identifies key facts, decisions, and preferences from the conversation"]], [Plain [Strong [Str "Conflict Resolution"], Str ": The system checks existing memories for duplicates or contradictions, ensuring the latest truth wins"]], [Plain [Strong [Str "Storage"], Str ": Memories are stored in vector storage (plus optional graph storage) for retrieval"]]], Header 3 ("graph-memory", ["unnumbered", "unlisted"], []) [Str "Graph Memory"], Para [Str "Graph Memory augments the standard vector search by automatically establishing connections between entities in stored data. When enabled, the system extracts entities (people, locations, jobs) and determines their relationships ", Str "[", Str "4", Str "]", Str ":"], BulletList [[Plain [Str "Vector search returns top semantic matches with optional reranking"]], [Plain [Str "Graph relations are returned alongside vector results to provide additional context"]], [Plain [Str "Entity relationships include source, target, relationship type, and confidence scores"]]], Para [Str "Graph memory is enabled per-call with ", Code ("", [], []) "enable_graph=True", Str " or globally at the project level ", Str "[", Str "4", Str "]", Str "."], Header 3 ("search-pipeline", ["unnumbered", "unlisted"], []) [Str "Search Pipeline"], Para [Str "Memory retrieval follows four stages ", Str "[", Str "7", Str "]", Str ":"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Query Processing"], Str ": Natural-language questions are cleaned and enriched for embedding search"]], [Plain [Strong [Str "Vector Search"], Str ": Embeddings locate closest memories via cosine similarity"]], [Plain [Strong [Str "Filtering and Reranking"], Str ": Logical filters (AND/OR, comparison operators) narrow candidates; optional rerankers refine ordering"]], [Plain [Strong [Str "Results Delivery"], Str ": Formatted memories with metadata, timestamps, and relevance scores are returned"]]], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Mem0's architecture consists of a memory processing engine layered over configurable storage backends:"], CodeBlock ("", [""], []) "┌───────────────────────────────────────────┐
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
", Para [Str "The Platform version manages all infrastructure components (vector stores, graph services, rerankers) as a hosted service. The Open Source version requires users to configure and run each component ", Str "[", Str "2", Str "]", Str "[", Str "3", Str "]", Str "."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Automatic Memory Extraction"], Str ": LLM-powered extraction of key facts, preferences, and decisions from conversations with conflict resolution against existing memories"]], [Plain [Strong [Str "Semantic Search"], Str ": Natural language queries with cosine similarity matching, optional reranking, and configurable similarity thresholds"]], [Plain [Strong [Str "Graph Memory"], Str ": Relationship-aware recall that extracts entities and their connections, returning graph relations alongside vector search results"]], [Plain [Strong [Str "Four Memory Layers"], Str ": Conversation, session, user, and organizational memory with automatic promotion and hierarchical retrieval"]], [Plain [Strong [Str "Metadata Filtering"], Str ": JSON-based logical filters (AND/OR, comparison operators) for date ranges, categories, and custom metadata"]], [Plain [Strong [Str "Multi-Framework Integration"], Str ": Native support for LangChain, CrewAI, Vercel AI SDK, AutoGen, LlamaIndex, LangGraph, and 15+ other frameworks"]], [Plain [Strong [Str "MCP Support"], Str ": Model Context Protocol (MCP) server for universal AI client connectivity"]], [Plain [Strong [Str "Async-by-Default"], Str ": Asynchronous client support since v1.0.0 for high-throughput applications"]], [Plain [Strong [Str "Reranking"], Str ": Configurable reranker support (Cohere, Zero Entropy) for improved retrieval precision"]], [Plain [Strong [Str "Multimodal Support"], Str ": Memory operations supporting multiple content modalities"]], [Plain [Strong [Str "Custom Categories"], Str ": User-defined categorization for organizing and filtering memories"]], [Plain [Strong [Str "Webhook Integrations"], Str ": Event-driven notifications for memory operations"]], [Plain [Strong [Str "Enterprise Controls"], Str ": SOC 2 Type II compliance, GDPR adherence, audit logs, and workspace governance (Platform)"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Personal AI Assistants"], Str ": Building agents that remember user preferences, dietary restrictions, travel plans, and conversation history across sessions"]], [Plain [Strong [Str "Customer Support"], Str ": Agents accumulating institutional knowledge and tracking customer histories to avoid repetitive questions"]], [Plain [Strong [Str "Multi-Agent Coordination"], Str ": Shared organizational memory enabling teams of agents to access consistent context"]], [Plain [Strong [Str "Onboarding Workflows"], Str ": Session memory tracking multi-step processes with bounded timeframes"]], [Plain [Strong [Str "Recommendation Systems"], Str ": Storing and retrieving user preferences for personalized suggestions (movies, restaurants, products)"]], [Plain [Strong [Str "Voice Agents"], Str ": Integration with LiveKit, ElevenLabs, and Pipecat for conversational AI with persistent memory"]], [Plain [Strong [Str "RAG Enhancement"], Str ": Augmenting Retrieval-Augmented Generation (RAG) pipelines with persistent user context"]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("add-memory", ["unnumbered", "unlisted"], []) [Str "Add Memory"], CodeBlock ("", ["python"], []) "from mem0 import MemoryClient

client = MemoryClient(api_key=\"your-api-key\")

messages = [
    {\"role\": \"user\", \"content\": \"I'm planning a trip to Tokyo next month.\"},
    {\"role\": \"assistant\", \"content\": \"Great! I'll remember that for future suggestions.\"}
]

# Platform
result = client.add(messages=messages, user_id=\"alice\")

# With metadata and graph
result = client.add(
    messages=messages,
    user_id=\"alice\",
    metadata={\"category\": \"travel\"},
    enable_graph=True,
)
", Header 3 ("search-memory", ["unnumbered", "unlisted"], []) [Str "Search Memory"], CodeBlock ("", ["python"], []) "# Basic search
results = client.search(\"What do you know about me?\", filters={\"user_id\": \"alice\"})

# With advanced filters (Platform)
results = client.search(
    \"hotel preferences\",
    filters={
        \"AND\": [
            {\"user_id\": \"alice\"},
            {\"categories\": {\"contains\": \"travel\"}},
        ]
    },
)

# Open Source
from mem0 import Memory
m = Memory()
results = m.search(\"hotel preferences\", user_id=\"alice\")
", Header 3 ("update-memory", ["unnumbered", "unlisted"], []) [Str "Update Memory"], CodeBlock ("", ["python"], []) "client.update(memory_id=\"mem-id-123\", data=\"Updated preference: boutique hotels in Shibuya\")
", Header 3 ("delete-memory", ["unnumbered", "unlisted"], []) [Str "Delete Memory"], CodeBlock ("", ["python"], []) "# Delete specific memory
client.delete(memory_id=\"mem-id-123\")

# Delete all memories for a user
client.delete_all(filters={\"user_id\": \"alice\"})
", Header 3 ("get-all-memories", ["unnumbered", "unlisted"], []) [Str "Get All Memories"], CodeBlock ("", ["python"], []) "# Platform
memories = client.get_all(filters={\"AND\": [{\"user_id\": \"alice\"}]})

# Open Source
memories = m.get_all(user_id=\"alice\")
", Header 3 ("rest-api", ["unnumbered", "unlisted"], []) [Str "REST API"], CodeBlock ("", ["bash"], []) "# Add memory
curl -X POST https://api.mem0.ai/v1/memories/add \\
  -H \"Authorization: Token your-api-key\" \\
  -H \"Content-Type: application/json\" \\
  -d '{\"messages\": [{\"role\": \"user\", \"content\": \"I love sushi\"}], \"user_id\": \"alice\"}'

# Search memory
curl -X POST https://api.mem0.ai/v1/memories/search \\
  -H \"Authorization: Token your-api-key\" \\
  -H \"Content-Type: application/json\" \\
  -d '{\"query\": \"food preferences\", \"filters\": {\"user_id\": \"alice\"}}'
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("open-source-configuration", ["unnumbered", "unlisted"], []) [Str "Open Source Configuration"], Para [Str "Mem0 OSS uses a dictionary-based configuration system with four configurable components: vector stores, LLMs, embedders, and rerankers ", Str "[", Str "9", Str "]", Str ":"], CodeBlock ("", ["python"], []) "from mem0 import Memory

config = {
    \"llm\": {
        \"provider\": \"openai\",
        \"config\": {
            \"model\": \"gpt-4.1-mini\",
            \"temperature\": 0.2,
        }
    },
    \"embedder\": {
        \"provider\": \"ollama\",
        \"config\": {
            \"model\": \"nomic-embed-text\",
        }
    },
    \"vector_store\": {
        \"provider\": \"qdrant\",
        \"config\": {
            \"collection_name\": \"my_memories\",
            \"host\": \"localhost\",
            \"port\": 6333,
        }
    },
}

m = Memory.from_config(config)
# Or from file: m = Memory.from_config_file(\"config.yaml\")
", Header 3 ("supported-providers", ["unnumbered", "unlisted"], []) [Str "Supported Providers"], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Component"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Providers"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "LLM"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "OpenAI, Azure OpenAI, Anthropic, Ollama, local models"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Embedder"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "OpenAI, Vertex AI, Ollama, Cohere"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Vector Store"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Qdrant, PostgreSQL (pgvector), managed alternatives"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Graph Store"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Neo4j, Memgraph"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Reranker"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Cohere, Zero Entropy"]]]])] (TableFoot ("", [], []) []), Header 3 ("configuration-best-practices", ["unnumbered", "unlisted"], []) [Str "Configuration Best Practices"], BulletList [[Plain [Str "Keep extraction temperatures at 0.2 or below for deterministic memory processing"]], [Plain [Str "Limit reranker ", Code ("", [], []) "top_k", Str " to 10-20 results"]], [Plain [Str "Name vector collections explicitly in production for tenant isolation"]], [Plain [Str "Store API credentials in environment variables ", Str "[", Str "9", Str "]"]]], Header 3 ("platform-configuration", ["unnumbered", "unlisted"], []) [Str "Platform Configuration"], Para [Str "The Platform manages infrastructure automatically. Configuration is done at the project level:"], CodeBlock ("", ["python"], []) "# Enable graph memory for all operations
client.project.update(enable_graph=True)
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("langchain-integration", ["unnumbered", "unlisted"], []) [Str "LangChain Integration"], CodeBlock ("", ["python"], []) "from langchain.memory import Mem0Memory

memory = Mem0Memory(api_key=\"your-key\", user_id=\"alice\")
# Use as LangChain memory backend
", Header 3 ("crewai-integration", ["unnumbered", "unlisted"], []) [Str "CrewAI Integration"], Para [Str "Mem0 serves as a shared memory backend for CrewAI agent crews, enabling persistent context across collaborative agent interactions ", Str "[", Str "8", Str "]", Str "."], Header 3 ("mcp-integration", ["unnumbered", "unlisted"], []) [Str "MCP Integration"], Para [Str "Mem0 provides an MCP server for universal AI client connectivity, enabling any MCP-compatible client to manage memory autonomously ", Str "[", Str "2", Str "]", Str "."], Header 3 ("vercel-ai-sdk", ["unnumbered", "unlisted"], []) [Str "Vercel AI SDK"], Para [Str "Integration with the Vercel AI SDK enables memory-powered applications in Next.js and other JavaScript frameworks ", Str "[", Str "8", Str "]", Str "."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("personalized-assistant-with-memory", ["unnumbered", "unlisted"], []) [Str "Personalized Assistant with Memory"], CodeBlock ("", ["python"], []) "from mem0 import MemoryClient

client = MemoryClient(api_key=\"your-api-key\")

# Store user preferences from conversation
messages = [
    {\"role\": \"user\", \"content\": \"I'm vegetarian and allergic to nuts.\"},
    {\"role\": \"assistant\", \"content\": \"I've noted your dietary preferences.\"},
    {\"role\": \"user\", \"content\": \"I prefer boutique hotels over large chains.\"},
    {\"role\": \"assistant\", \"content\": \"Got it! Boutique hotels for your travels.\"},
]
client.add(messages=messages, user_id=\"alice\")

# Later, retrieve relevant context
results = client.search(\"restaurant suggestions\", filters={\"user_id\": \"alice\"})
# Returns: memories about vegetarian preference and nut allergy
", Header 3 ("graph-enhanced-memory", ["unnumbered", "unlisted"], []) [Str "Graph-Enhanced Memory"], CodeBlock ("", ["python"], []) "messages = [
    {\"role\": \"user\", \"content\": \"My name is Joseph. I'm from Seattle and work as a software engineer.\"},
]
client.add(messages, user_id=\"joseph\", enable_graph=True)

# Search returns both vector matches and entity relationships
results = client.search(\"what is my name?\", user_id=\"joseph\", enable_graph=True)
# Results include relations: joseph -> lives_in -> Seattle, joseph -> works_as -> software_engineer
", Header 3 ("multi-session-context", ["unnumbered", "unlisted"], []) [Str "Multi-Session Context"], CodeBlock ("", ["python"], []) "# Session 1: Trip planning
client.add(
    [{\"role\": \"user\", \"content\": \"I want to visit Tokyo in March.\"}],
    user_id=\"alex\",
    session_id=\"trip-planning-2025\",
)

# Session 2: Different context, same user memory
results = client.search(
    \"Any travel plans?\",
    user_id=\"alex\",
)
# Returns Tokyo trip memory from previous session
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "OpenAI dependency"], Str ": Default open source configuration requires an OpenAI API key; alternative providers require explicit configuration ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Async processing for graph"], Str ": Adding memories with graph enabled is asynchronous; memories may not be immediately available for retrieval ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "Inference mode mixing"], Str ": Using both ", Code ("", [], []) "infer=True", Str " and ", Code ("", [], []) "infer=False", Str " for identical content creates duplicate memories ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "Date filtering"], Str ": Date range filters are available only on the Platform, not in the open source version ", Str "[", Str "7", Str "]"]], [Plain [Strong [Str "Security consideration"], Str ": The retrieval-by-design architecture means stored content is accessible by any query scoped to the same identifiers; avoid storing unencrypted secrets or personally identifiable information ", Str "[", Str "5", Str "]"]], [Plain [Strong [Str "Graph memory overhead"], Str ": Graph processing introduces additional latency, though generally acceptable for most use cases ", Str "[", Str "4", Str "]"]], [Plain [Strong [Str "Reranking disabled by default"], Str ": Must be explicitly configured for improved retrieval precision ", Str "[", Str "3", Str "]"]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "v1.0.0"], Str ": Major release shipping rerankers, async-by-default behavior, Azure OpenAI support, and breaking API changes ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Pre-v1.0"], Str ": Initial releases establishing core memory operations, vector search, and Python/JavaScript SDK support"]], [Plain [Strong [Str "Graph Memory"], Str ": Added relationship-aware recall with Neo4j and Memgraph support"]], [Plain [Strong [Str "OpenMemory"], Str ": Introduced workspace-based memory for multi-agent team collaboration"]], [Plain [Strong [Str "MCP Support"], Str ": Added Model Context Protocol server for universal AI client integration"]], [Plain [Strong [Str "Multimodal"], Str ": Added support for multimodal memory content"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Welcome to Mem0 - https://docs.mem0.ai/introduction"]], [Plain [Str "[", Str "2", Str "]", Str " Platform Overview - https://docs.mem0.ai/platform/overview"]], [Plain [Str "[", Str "3", Str "]", Str " Open Source Overview - https://docs.mem0.ai/open-source/overview"]], [Plain [Str "[", Str "4", Str "]", Str " Graph Memory - https://docs.mem0.ai/platform/features/graph-memory"]], [Plain [Str "[", Str "5", Str "]", Str " Memory Types - https://docs.mem0.ai/core-concepts/memory-types"]], [Plain [Str "[", Str "6", Str "]", Str " Add Memory - https://docs.mem0.ai/core-concepts/memory-operations/add"]], [Plain [Str "[", Str "7", Str "]", Str " Search Memory - https://docs.mem0.ai/core-concepts/memory-operations/search"]], [Plain [Str "[", Str "8", Str "]", Str " Integrations - https://docs.mem0.ai/integrations"]], [Plain [Str "[", Str "9", Str "]", Str " Configuration - https://docs.mem0.ai/open-source/configuration"]]]]