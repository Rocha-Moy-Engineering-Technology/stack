[Header 1 ("zep", [], []) [Str "Zep"], BlockQuote [Para [Str "Context engineering platform with temporal knowledge graphs and agent memory for personalization"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Name"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Zep"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Memory Systems"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API/SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "no"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Docs"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "help.getzep.com"] ("https://help.getzep.com/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Zep is a context engineering platform that systematically assembles personalized context — user preferences, traits, and business data — for reliable agent applications. It combines agent memory, Graph Retrieval-Augmented Generation (RAG), and context assembly capabilities to deliver comprehensive personalized context that reduces hallucinations and improves accuracy ", Str "[", Str "1", Str "]", Str "."], Para [Str "The platform's distinguishing feature is its ", Strong [Str "temporal knowledge graph"], Str ", where nodes represent entities and edges represent facts and relationships that update dynamically. Facts carry temporal validity markers, tracking when information became valid and when it was invalidated by newer data ", Str "[", Str "2", Str "]", Str "."], Para [Str "Zep provides SDKs for Python, TypeScript, and Go, and integrates with agent frameworks including LangGraph, AutoGen, and CrewAI. The platform includes enterprise features such as HIPAA compliance, role-based access control (RBAC), and audit logging ", Str "[", Str "1", Str "]", Str "."], Para [Str "Zep's underlying graph technology is built on ", Strong [Str "Graphiti"], Str ", an open-source temporal knowledge graph framework (23,000+ GitHub stars) maintained by the Zep team ", Str "[", Str "1", Str "]", Str "."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Header 3 ("temporal-knowledge-graphs", ["unnumbered", "unlisted"], []) [Str "Temporal Knowledge Graphs"], Para [Str "Zep's knowledge graph is its unified knowledge store. Nodes represent entities (people, places, concepts) and edges represent facts and relationships between them. A key temporal feature is ", Strong [Str "fact invalidation"], Str ": when new information supersedes prior knowledge, the system records when the old fact became invalid on that fact's edge, preserving the full temporal history ", Str "[", Str "2", Str "]", Str "."], Header 3 ("graph-types", ["unnumbered", "unlisted"], []) [Str "Graph Types"], Para [Str "Zep supports two graph structures ", Str "[", Str "2", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Graph"], Str ": An arbitrary knowledge graph for storing current knowledge about objects or systems"]], [Plain [Strong [Str "User Graph"], Str ": A specialized graph for preserving personalized context specific to individual application users. All messages added to any thread of that user are ingested into the user's graph by default"]]], Header 3 ("users-and-threads", ["unnumbered", "unlisted"], []) [Str "Users and Threads"], Para [Strong [Str "Users"], Str " represent individual application users, each with their own knowledge graph. Users should be created with at minimum a first name, ideally including last name and email for accurate entity identification in the graph ", Str "[", Str "3", Str "]", Str "."], Para [Strong [Str "Threads"], Str " represent conversation threads belonging to a user. Messages added to any thread are automatically ingested into that user's knowledge graph. Threads are identified by unique IDs and associated with a user ID ", Str "[", Str "3", Str "]", Str "."], Header 3 ("context-blocks", ["unnumbered", "unlisted"], []) [Str "Context Blocks"], Para [Str "A ", Strong [Str "Context Block"], Str " is an optimized string containing a user summary and facts from the knowledge graph most relevant to the current thread. It includes dates when facts became valid and invalid. Context blocks are retrieved via ", Code ("", [], []) "thread.get_user_context()", Str " and combine semantic search, full-text search, and breadth-first search for comprehensive retrieval ", Str "[", Str "2", Str "]", Str "[", Str "3", Str "]", Str "."], Header 3 ("data-types", ["unnumbered", "unlisted"], []) [Str "Data Types"], Para [Str "Zep ingests multiple data formats ", Str "[", Str "2", Str "]", Str "[", Str "3", Str "]", Str ":"], BulletList [[Plain [Strong [Str "Messages"], Str ": Chat history with user/assistant roles and timestamps"]], [Plain [Strong [Str "JSON"], Str ": Structured business data (transactions, events, user interactions)"]], [Plain [Strong [Str "Text"], Str ": Unstructured documents, emails, and support tickets"]]], Header 3 ("context-templates", ["unnumbered", "unlisted"], []) [Str "Context Templates"], Para [Str "Custom context templates allow developers to control the structure of retrieved context using template variables like ", Code ("", [], []) "%{user_summary}", Str ", ", Code ("", [], []) "%{edges limit=10}", Str ", and ", Code ("", [], []) "%{entities limit=5}", Str ". Templates are created once and referenced by ID when retrieving context ", Str "[", Str "3", Str "]", Str "."], Header 2 ("installation", ["unnumbered", "unlisted"], []) [Str "Installation"], Header 3 ("python", ["unnumbered", "unlisted"], []) [Str "Python"], CodeBlock ("", ["bash"], []) "pip install zep-cloud
# or with uv
uv pip install zep-cloud
", Header 3 ("typescript", ["unnumbered", "unlisted"], []) [Str "TypeScript"], CodeBlock ("", ["bash"], []) "npm install @getzep/zep-cloud
# or
yarn add @getzep/zep-cloud
# or
pnpm install @getzep/zep-cloud
", Header 3 ("go", ["unnumbered", "unlisted"], []) [Str "Go"], CodeBlock ("", ["bash"], []) "go get github.com/getzep/zep-go/v3
", Header 3 ("client-initialization", ["unnumbered", "unlisted"], []) [Str "Client Initialization"], CodeBlock ("", ["python"], []) "import os
from zep_cloud.client import Zep

client = Zep(api_key=os.environ.get(\"ZEP_API_KEY\"))
", Para [Str "API keys are obtained from the Zep dashboard at ", Link ("", [], []) [Str "app.getzep.com"] ("https://app.getzep.com/", ""), Str " ", Str "[", Str "3", Str "]", Str "."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Zep's architecture centers on a temporal knowledge graph that ingests data from multiple sources and assembles personalized context for agent consumption:"], CodeBlock ("", [""], []) "┌─────────────────────────────────────────────┐
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
", Para [Str "The platform retrieves context in sub-200ms latency, optimizing for ", Strong [Str "high recall over precision"], Str " — preferring inclusion of more results even if some are less relevant ", Str "[", Str "3", Str "]", Str "."], Header 2 ("key-features", ["unnumbered", "unlisted"], []) [Str "Key Features"], BulletList [[Plain [Strong [Str "Temporal Knowledge Graphs"], Str ": Dynamic graph with nodes (entities) and edges (facts) that track temporal validity, including when facts become valid and invalid"]], [Plain [Strong [Str "Automatic Fact Extraction"], Str ": Ingested messages and data are automatically processed to extract entities, relationships, and facts into the knowledge graph"]], [Plain [Strong [Str "Fact Invalidation"], Str ": When new information supersedes prior knowledge, the old fact's invalidation time is preserved on the graph edge"]], [Plain [Strong [Str "Context Assembly"], Str ": Optimized context blocks combining user summaries and relevant facts with temporal validity markers, retrieved in sub-200ms"]], [Plain [Strong [Str "Custom Context Templates"], Str ": Configurable templates for controlling context structure using variables like ", Code ("", [], []) "%{user_summary}", Str ", ", Code ("", [], []) "%{edges}", Str ", ", Code ("", [], []) "%{entities}"]], [Plain [Strong [Str "Multi-Language SDKs"], Str ": Native SDKs for Python, TypeScript, and Go with consistent APIs"]], [Plain [Strong [Str "User Graphs"], Str ": Per-user knowledge graphs that automatically ingest all thread messages for personalization"]], [Plain [Strong [Str "Business Data Ingestion"], Str ": Support for JSON, text, and message data types representing transactions, events, documents, and emails"]], [Plain [Strong [Str "Batch Ingestion"], Str ": Bulk data loading for backfilling existing users and conversations"]], [Plain [Strong [Str "Graph RAG"], Str ": Graph-based retrieval augmented generation combining semantic search, full-text search, and breadth-first graph search"]], [Plain [Strong [Str "Agentic Tools"], Str ": Tool definitions enabling agents to directly query user knowledge graphs"]], [Plain [Strong [Str "Custom Entity and Edge Types"], Str ": Pydantic-like class definitions for specialized graph structures"]], [Plain [Strong [Str "HIPAA Compliance"], Str ": Enterprise-grade healthcare data compliance"]], [Plain [Strong [Str "RBAC and Audit Logging"], Str ": Role-based access control with comprehensive audit trails"]], [Plain [Strong [Str "Playground"], Str ": Web-based environment for testing graph queries and context retrieval"]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Personalized AI Assistants"], Str ": Building agents that remember user preferences, traits, and history across conversations with temporal awareness"]], [Plain [Strong [Str "Customer Support"], Str ": Agents with access to customer interaction history, support tickets, and product knowledge via the knowledge graph"]], [Plain [Strong [Str "Healthcare Applications"], Str ": HIPAA-compliant memory for medical AI assistants tracking patient interactions and preferences"]], [Plain [Strong [Str "E-Commerce Personalization"], Str ": Ingesting purchase history, browsing behavior, and preferences as structured JSON data for recommendation agents"]], [Plain [Strong [Str "Music and Content Recommendation"], Str ": Tracking user listening/viewing behavior and extracting preference patterns through entity relationships"]], [Plain [Strong [Str "Multi-Agent Systems"], Str ": Shared knowledge graphs enabling context continuity across different specialized agents"]], [Plain [Strong [Str "Enterprise Knowledge Management"], Str ": Organizational graphs storing company-wide policies, procedures, and institutional knowledge"]]], Header 2 ("api-reference", ["unnumbered", "unlisted"], []) [Str "API Reference"], Header 3 ("user-management", ["unnumbered", "unlisted"], []) [Str "User Management"], CodeBlock ("", ["python"], []) "# Create user
user = client.user.add(
    user_id=\"internal_id\",
    email=\"jane@example.com\",
    first_name=\"Jane\",
    last_name=\"Smith\",
)

# Get user
user = client.user.get(user_id=\"internal_id\")
", Header 3 ("thread-operations", ["unnumbered", "unlisted"], []) [Str "Thread Operations"], CodeBlock ("", ["python"], []) "import uuid
from zep_cloud.types import Message
from datetime import datetime, timezone

# Create thread
thread_id = uuid.uuid4().hex
client.thread.create(thread_id=thread_id, user_id=user_id)

# Add messages (include name and RFC3339 timestamp)
messages = [
    Message(
        created_at=datetime.now(timezone.utc).isoformat(),
        name=\"Jane Smith\",
        role=\"user\",
        content=\"Who was Octavia Butler?\",
    )
]
response = client.thread.add_messages(thread_id, messages=messages)

# Add assistant response
assistant_messages = [
    Message(
        created_at=datetime.now(timezone.utc).isoformat(),
        name=\"AI Assistant\",
        role=\"assistant\",
        content=\"Octavia Butler was an influential American science fiction writer...\",
    )
]
client.thread.add_messages(thread_id, messages=assistant_messages)
", Header 3 ("context-retrieval", ["unnumbered", "unlisted"], []) [Str "Context Retrieval"], CodeBlock ("", ["python"], []) "# Default context block
user_context = client.thread.get_user_context(thread_id=thread_id)
context_block = user_context.context

# Custom template context
user_context = client.thread.get_user_context(
    thread_id=thread_id,
    template_id=\"customer-support\",
)
", Header 3 ("graph-data-ingestion", ["unnumbered", "unlisted"], []) [Str "Graph Data Ingestion"], CodeBlock ("", ["python"], []) "import json

# Add structured business data
event_data = {
    \"user_id\": \"user123\",
    \"user_name\": \"Jane Smith\",
    \"event_type\": \"song_played\",
    \"song_title\": \"Bohemian Rhapsody\",
    \"artist\": \"Queen\",
    \"duration_seconds\": 354,
}

client.graph.add(
    user_id=\"user123\",
    type=\"json\",
    data=json.dumps(event_data),
)
", Header 3 ("graph-search", ["unnumbered", "unlisted"], []) [Str "Graph Search"], CodeBlock ("", ["python"], []) "# Search the knowledge graph
results = client.graph.search(
    user_id=\"user123\",
    query=\"music preferences\",
)
", Header 3 ("context-templates-1", ["unnumbered", "unlisted"], []) [Str "Context Templates"], CodeBlock ("", ["python"], []) "# Create a custom template
client.context.create_context_template(
    template_id=\"customer-support\",
    template=\"\"\"# CUSTOMER PROFILE
%{user_summary}

# RECENT INTERACTIONS
%{edges limit=10}

# KEY ENTITIES
%{entities limit=5}\"\"\",
)
", Header 2 ("configuration", ["unnumbered", "unlisted"], []) [Str "Configuration"], Header 3 ("api-key-setup", ["unnumbered", "unlisted"], []) [Str "API Key Setup"], CodeBlock ("", ["bash"], []) "# Environment variable
export ZEP_API_KEY=your_api_key_here

# Or in .env file
ZEP_API_KEY=your_api_key_here
", Header 3 ("context-window-integration", ["unnumbered", "unlisted"], []) [Str "Context Window Integration"], Para [Str "Two recommended approaches for inserting Zep context into LLM calls ", Str "[", Str "3", Str "]", Str ":"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "System Prompt Injection"], Str ": Append the context block directly to the system prompt, refreshing dynamically on each turn"]], [Plain [Strong [Str "Context Message Approach"], Str ": Insert the context block as a tool message after user messages, which enables prompt caching for improved efficiency"]]], Header 3 ("timestamp-format", ["unnumbered", "unlisted"], []) [Str "Timestamp Format"], Para [Str "All messages should use RFC3339 timestamp format for accurate temporal understanding in the knowledge graph ", Str "[", Str "3", Str "]", Str "."], Header 3 ("user-names-in-messages", ["unnumbered", "unlisted"], []) [Str "User Names in Messages"], Para [Str "Including user names in messages is critical for accurate graph construction — the system uses names to identify and link entities in the knowledge graph ", Str "[", Str "3", Str "]", Str "."], Header 3 ("backfilling-existing-data", ["unnumbered", "unlisted"], []) [Str "Backfilling Existing Data"], Para [Str "For existing users and conversations, loop through data calling ", Code ("", [], []) "user.add", Str " and ", Code ("", [], []) "thread.add_messages", Str ", or use batch processing methods for large-scale ingestion ", Str "[", Str "3", Str "]", Str "."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("langgraph-integration", ["unnumbered", "unlisted"], []) [Str "LangGraph Integration"], Para [Str "Zep integrates with LangGraph for building complex agent workflows with persistent memory and knowledge graph access ", Str "[", Str "1", Str "]", Str "."], Header 3 ("autogen-integration", ["unnumbered", "unlisted"], []) [Str "AutoGen Integration"], Para [Str "Multi-agent AutoGen systems can leverage Zep for shared context and persistent memory across agent interactions ", Str "[", Str "1", Str "]", Str "."], Header 3 ("crewai-integration", ["unnumbered", "unlisted"], []) [Str "CrewAI Integration"], Para [Str "CrewAI agent crews can use Zep as a memory backend for maintaining context across collaborative agent tasks ", Str "[", Str "1", Str "]", Str "."], Header 3 ("mcp-server", ["unnumbered", "unlisted"], []) [Str "MCP Server"], Para [Str "Zep provides a Model Context Protocol (MCP) server and ", Code ("", [], []) "llms.txt", Str " file for connecting AI coding assistants directly to Zep's documentation and capabilities ", Str "[", Str "1", Str "]", Str "."], Header 3 ("agentic-tools", ["unnumbered", "unlisted"], []) [Str "Agentic Tools"], Para [Str "Zep provides tool definitions that enable agents to directly query user knowledge graphs during conversations, allowing agents to retrieve context autonomously ", Str "[", Str "2", Str "]", Str "."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-conversation-with-memory", ["unnumbered", "unlisted"], []) [Str "Basic Conversation with Memory"], CodeBlock ("", ["python"], []) "import os
import uuid
from zep_cloud.client import Zep
from zep_cloud.types import Message
from datetime import datetime, timezone

client = Zep(api_key=os.environ[\"ZEP_API_KEY\"])

# Create user
user = client.user.add(
    user_id=\"jane_123\",
    first_name=\"Jane\",
    last_name=\"Smith\",
    email=\"jane@example.com\",
)

# Create thread
thread_id = uuid.uuid4().hex
client.thread.create(thread_id=thread_id, user_id=\"jane_123\")

# Add conversation messages
messages = [
    Message(
        created_at=datetime.now(timezone.utc).isoformat(),
        name=\"Jane Smith\",
        role=\"user\",
        content=\"I'm vegetarian and I love Italian food.\",
    )
]
client.thread.add_messages(thread_id, messages=messages)

# Retrieve personalized context for future interactions
user_context = client.thread.get_user_context(thread_id=thread_id)
print(user_context.context)
# Output includes: user summary, dietary preferences, temporal validity dates
", Header 3 ("business-data-ingestion", ["unnumbered", "unlisted"], []) [Str "Business Data Ingestion"], CodeBlock ("", ["python"], []) "import json

# Ingest purchase history
purchase = {
    \"user_id\": \"jane_123\",
    \"user_name\": \"Jane Smith\",
    \"event_type\": \"purchase\",
    \"item\": \"Margherita Pizza Cookbook\",
    \"category\": \"books\",
    \"amount\": 24.99,
}

client.graph.add(
    user_id=\"jane_123\",
    type=\"json\",
    data=json.dumps(purchase),
)

# The knowledge graph now links Jane to Italian cooking interests
# Future context blocks will include this preference
", Header 3 ("custom-context-template", ["unnumbered", "unlisted"], []) [Str "Custom Context Template"], CodeBlock ("", ["python"], []) "# Define a support-focused template
client.context.create_context_template(
    template_id=\"support-agent\",
    template=\"\"\"# USER PROFILE
%{user_summary}

# RELEVANT HISTORY
%{edges limit=15}

# KEY TOPICS
%{entities limit=8}\"\"\",
)

# Retrieve context using the template
ctx = client.thread.get_user_context(
    thread_id=thread_id,
    template_id=\"support-agent\",
)

# Use in LLM system prompt
system_prompt = f\"You are a support agent.\\n\\n{ctx.context}\"
", Header 2 ("limitations", ["unnumbered", "unlisted"], []) [Str "Limitations"], BulletList [[Plain [Strong [Str "Closed source"], Str ": Zep is a managed service without a self-hosted open-source option (though Graphiti, the underlying graph framework, is open source)"]], [Plain [Strong [Str "API-dependent"], Str ": All operations require API connectivity to Zep's cloud infrastructure"]], [Plain [Strong [Str "High recall bias"], Str ": The platform optimizes for high recall over precision, which may return less relevant results alongside relevant ones ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Timestamp requirements"], Str ": RFC3339 format timestamps are required for accurate temporal tracking; missing or incorrect timestamps degrade temporal understanding ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Name dependency"], Str ": User names in messages are critical for accurate graph construction; anonymous messages may result in incomplete entity linking ", Str "[", Str "3", Str "]"]], [Plain [Strong [Str "Asynchronous graph processing"], Str ": Data ingestion into the knowledge graph is asynchronous; recently added data may not be immediately available in context blocks"]], [Plain [Strong [Str "No offline/local deployment"], Str ": Unlike competitors, Zep does not offer a fully self-hosted deployment option"]]], Header 2 ("changelog", ["unnumbered", "unlisted"], []) [Str "Changelog"], BulletList [[Plain [Strong [Str "v3"], Str ": Current major version featuring temporal knowledge graphs, context assembly engine, custom context templates, and Graph RAG"]], [Plain [Strong [Str "Graphiti"], Str ": Open-source temporal knowledge graph framework extracted from Zep's core technology (23,000+ GitHub stars, 2,300+ forks)"]], [Plain [Strong [Str "MCP Server"], Str ": Added Model Context Protocol server for AI coding assistant integration"]], [Plain [Strong [Str "Mem0 Migration"], Str ": Published migration guide for users transitioning from Mem0 to Zep"]], [Plain [Strong [Str "Multi-language SDKs"], Str ": Python, TypeScript, and Go SDKs with consistent API surfaces"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " Welcome to Zep - https://help.getzep.com/overview"]], [Plain [Str "[", Str "2", Str "]", Str " Key Concepts - https://help.getzep.com/concepts"]], [Plain [Str "[", Str "3", Str "]", Str " Quick Start Guide - https://help.getzep.com/quick-start-guide"]]]]