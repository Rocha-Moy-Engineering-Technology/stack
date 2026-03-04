[Header 1 ("langgraph", [], []) [Str "LangGraph"], BlockQuote [Para [Str "Low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents with durable execution, human-in-the-loop capabilities, and comprehensive memory support."]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.5)), (AlignDefault, (ColWidth 0.5))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Agent Frameworks"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "https://github.com/langchain-ai/langgraph"] ("https://github.com/langchain-ai/langgraph", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "24935"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.langchain.com/oss/python/langgraph/overview", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "LangGraph is a low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents. Part of the LangChain ecosystem, it provides graph-based workflow composition where agents are defined as nodes connected by edges, with shared state flowing through the execution graph. LangGraph is available in both Python and TypeScript, and it operates standalone or alongside LangChain and LangSmith for observability and evaluation."], Para [Str "Unlike higher-level agent frameworks that prescribe specific LLM patterns or tool-calling approaches, LangGraph provides the primitives for constructing arbitrary agent architectures. Developers define computational nodes, wire them together with conditional or direct edges, and let the framework handle state management, checkpointing, and durable execution. This low-level design makes LangGraph suitable for complex decision logic, extended execution tasks, failure recovery, and multi-agent systems that require fine-grained control over agent behavior."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Graphs"], Str " are the foundational abstraction in LangGraph. An agent workflow is represented as a directed graph where nodes perform computation and edges define transitions between them. The graph structure determines how state flows through the system and which nodes execute in which order."], Para [Strong [Str "Nodes"], Str " are the computational units within a graph. Each node is a function that receives the current state, performs some operation (such as calling an LLM, executing a tool, or transforming data), and returns an updated state. Nodes encapsulate discrete steps of agent logic."], Para [Strong [Str "Edges"], Str " define transitions between nodes. They can be direct (always transition from node A to node B) or conditional (evaluate a predicate on the current state to determine which node executes next). Conditional edges enable branching logic, loops, and dynamic routing within agent workflows."], Para [Strong [Str "State"], Str " is a shared data structure that flows through the graph. It accumulates information as nodes process it, serving as the working context for the entire agent execution. State is typed and structured, providing a predictable contract between nodes."], Para [Strong [Str "Channels"], Str " enable stateful data flow within the graph. They define how state updates are merged and propagated, supporting patterns like message accumulation, key-value storage, and append-only logs."], Para [Strong [Str "Checkpointing"], Str " provides durable execution by persisting agent state at defined points. If an agent fails or is interrupted, execution resumes from the most recent checkpoint rather than restarting from scratch. This is critical for long-running workflows that interact with external systems."], Para [Strong [Str "Human-in-the-Loop"], Str " support allows developers to inspect and modify agent state at any point during execution. Agents can pause, present their current state for human review or correction, and then resume with the modified state. This enables supervised agent deployments where human oversight is required."], Para [Strong [Str "Memory"], Str " in LangGraph spans two dimensions: short-term working memory within a single session and persistent long-term memory that carries across sessions. This dual-memory model allows agents to maintain context within a conversation while also recalling information from previous interactions."], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Para [Str "Install LangGraph using pip or uv:"], CodeBlock ("", ["bash"], []) "pip install -U langgraph
", CodeBlock ("", ["bash"], []) "uv add langgraph
", Para [Str "A minimal agent graph can be constructed as follows:"], CodeBlock ("", ["python"], []) "from langgraph.graph import StateGraph, START, END
from typing import TypedDict

class AgentState(TypedDict):
    messages: list[str]
    result: str

def process_input(state: AgentState) -> dict:
    return {\"result\": f\"Processed: {state['messages'][-1]}\"}

graph = StateGraph(AgentState)
graph.add_node(\"process\", process_input)
graph.add_edge(START, \"process\")
graph.add_edge(\"process\", END)

app = graph.compile()
result = app.invoke({\"messages\": [\"Hello\"], \"result\": \"\"})
", Para [Str "For checkpointing with persistent storage:"], CodeBlock ("", ["python"], []) "from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()
app = graph.compile(checkpointer=memory)

config = {\"configurable\": {\"thread_id\": \"session-1\"}}
result = app.invoke({\"messages\": [\"Hello\"], \"result\": \"\"}, config)
", Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "LangGraph follows a graph-based orchestration architecture with no mandated LLM patterns or tool-calling approaches. The core runtime executes a directed graph where:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "State initialization"], Str " creates the shared data structure that flows through all nodes"]], [Plain [Strong [Str "Node execution"], Str " processes state through computational units in topological order, respecting edge conditions"]], [Plain [Strong [Str "Edge evaluation"], Str " determines the next node(s) to execute based on direct connections or conditional predicates"]], [Plain [Strong [Str "Checkpoint persistence"], Str " saves state at configurable points for durability"]], [Plain [Strong [Str "Completion"], Str " occurs when execution reaches a terminal node or an explicit end condition"]]], Para [Str "The architecture separates the graph definition (compile-time) from graph execution (runtime). Graphs are defined declaratively, compiled into an executable form, and then invoked with initial state. This separation allows the same graph definition to be executed with different checkpointing backends, memory configurations, and deployment targets."], Para [Str "LangGraph operates independently but integrates with the broader LangChain ecosystem:"], BulletList [[Plain [Strong [Str "Standalone"], Str ": Pure graph-based orchestration without LangChain dependencies"]], [Plain [Strong [Str "With LangChain"], Str ": Access to pre-built agent abstractions, tool integrations, and prompt management"]], [Plain [Strong [Str "With LangSmith"], Str ": Observability, tracing, evaluation, and monitoring of agent executions"]], [Plain [Strong [Str "With Agent Server"], Str ": Deployment platform for stateful agent workflows with scaling infrastructure"]]], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Durable Execution"], Str " ensures agents persist through failures and infrastructure disruptions. By checkpointing state at defined intervals, LangGraph enables agents to resume from the last successful point rather than restarting entirely. This is essential for workflows that span minutes, hours, or days."], Para [Strong [Str "Human-in-the-Loop Control"], Str " provides mechanisms to pause agent execution, surface the current state for human inspection or modification, and resume with updated state. This supports use cases where regulatory compliance, safety constraints, or domain expertise require human oversight at critical decision points."], Para [Strong [Str "Comprehensive Memory System"], Str " offers both short-term working memory within a session and persistent long-term memory across sessions. Agents can maintain conversational context, remember user preferences, and accumulate knowledge over time without external memory infrastructure."], Para [Strong [Str "Conditional Routing"], Str " through conditional edges enables dynamic agent behavior. Based on the current state, agents can branch into different execution paths, retry failed operations, or loop through iterative refinement steps. This goes beyond linear pipelines to support genuinely adaptive agent logic."], Para [Strong [Str "Multi-Agent Composition"], Str " allows multiple agent graphs to be composed into larger systems. Individual agents can be nodes within a parent graph, enabling hierarchical agent architectures where supervisor agents delegate to specialized sub-agents."], Para [Strong [Str "Production Deployment"], Str " via Agent Server provides scalable infrastructure for running stateful agent workflows. This includes horizontal scaling, persistent state management, and operational tooling for monitoring agent health and performance."], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Extended execution tasks"], Str ": Multi-step research, data processing pipelines, and workflows that run for extended periods and must survive infrastructure disruptions"]], [Plain [Strong [Str "Complex decision logic"], Str ": Agents that require branching, looping, and conditional routing based on intermediate results"]], [Plain [Strong [Str "Human oversight workflows"], Str ": Regulated industries or high-stakes decisions where human review is required before agent actions are finalized"]], [Plain [Strong [Str "Failure recovery"], Str ": Long-running processes that interact with unreliable external systems and need checkpoint-based recovery"]], [Plain [Strong [Str "Multi-agent systems"], Str ": Architectures where multiple specialized agents collaborate, with a supervisor agent orchestrating their interactions"]], [Plain [Strong [Str "Conversational agents with memory"], Str ": Chatbots and assistants that maintain context within and across sessions"]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Para [Strong [Str "Graph Construction"], Str ":"], BulletList [[Plain [Code ("", [], []) "StateGraph(state_schema)", Str " -- Create a new graph with a typed state schema"]], [Plain [Code ("", [], []) "graph.add_node(name, function)", Str " -- Add a computational node to the graph"]], [Plain [Code ("", [], []) "graph.add_edge(source, target)", Str " -- Add a direct edge between nodes"]], [Plain [Code ("", [], []) "graph.add_conditional_edges(source, condition_fn, mapping)", Str " -- Add conditional routing from a node"]], [Plain [Code ("", [], []) "graph.compile(checkpointer=None)", Str " -- Compile the graph into an executable application"]]], Para [Strong [Str "Execution"], Str ":"], BulletList [[Plain [Code ("", [], []) "app.invoke(initial_state, config)", Str " -- Execute the graph synchronously with initial state"]], [Plain [Code ("", [], []) "app.ainvoke(initial_state, config)", Str " -- Execute the graph asynchronously"]], [Plain [Code ("", [], []) "app.stream(initial_state, config)", Str " -- Stream execution results as they are produced"]]], Para [Strong [Str "Built-in References"], Str ":"], BulletList [[Plain [Code ("", [], []) "START", Str " -- Special node representing the graph entry point"]], [Plain [Code ("", [], []) "END", Str " -- Special node representing the graph terminal point"]]], Para [Strong [Str "Checkpointing"], Str ":"], BulletList [[Plain [Code ("", [], []) "MemorySaver()", Str " -- In-memory checkpoint storage for development"]], [Plain [Code ("", [], []) "SqliteSaver(connection)", Str " -- SQLite-backed checkpoint storage"]], [Plain [Code ("", [], []) "PostgresSaver(connection)", Str " -- PostgreSQL-backed checkpoint storage"]]], Para [Strong [Str "Configuration"], Str ":"], BulletList [[Plain [Code ("", [], []) "thread_id", Str " -- Identifies a conversation or execution thread for checkpointing"]], [Plain [Code ("", [], []) "checkpoint_id", Str " -- References a specific checkpoint for resumption"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Para [Str "LangGraph configuration is primarily code-driven through the graph compilation step and invocation config:"], CodeBlock ("", ["python"], []) "# Checkpointer configuration
from langgraph.checkpoint.memory import MemorySaver
checkpointer = MemorySaver()
app = graph.compile(checkpointer=checkpointer)

# Thread configuration for state persistence
config = {
    \"configurable\": {
        \"thread_id\": \"user-session-123\",
    }
}

# Invoke with configuration
result = app.invoke(initial_state, config)
", Para [Str "For production deployments, Agent Server configuration handles scaling, persistence backends, and operational parameters separately from the graph definition."], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Para [Strong [Str "LangChain Integration"], Str ": LangGraph works with LangChain's chat models, tools, and prompt templates. LangChain provides higher-level abstractions that can be used within LangGraph nodes:"], CodeBlock ("", ["python"], []) "from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph

llm = ChatOpenAI(model=\"gpt-4\")

def call_model(state):
    response = llm.invoke(state[\"messages\"])
    return {\"messages\": state[\"messages\"] + [response]}
", Para [Strong [Str "LangSmith Observability"], Str ": When LangSmith is configured, LangGraph executions are automatically traced, providing visibility into node-level execution times, state transitions, and LLM calls."], Para [Strong [Str "Tool Integration"], Str ": LangGraph nodes can invoke any callable, including LangChain tools, custom functions, API clients, and database operations. There is no restriction on what a node can execute."], Para [Strong [Str "External State Stores"], Str ": Checkpointing backends can be swapped between in-memory (development), SQLite (single-node), and PostgreSQL (production) without changing graph logic."], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Para [Strong [Str "Conditional routing based on agent state"], Str ":"], CodeBlock ("", ["python"], []) "from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Literal

class State(TypedDict):
    query: str
    category: str
    response: str

def classify(state: State) -> dict:
    if \"price\" in state[\"query\"].lower():
        return {\"category\": \"pricing\"}
    return {\"category\": \"general\"}

def handle_pricing(state: State) -> dict:
    return {\"response\": \"Routing to pricing information.\"}

def handle_general(state: State) -> dict:
    return {\"response\": \"Routing to general support.\"}

def route(state: State) -> Literal[\"pricing\", \"general\"]:
    return state[\"category\"]

graph = StateGraph(State)
graph.add_node(\"classify\", classify)
graph.add_node(\"pricing\", handle_pricing)
graph.add_node(\"general\", handle_general)

graph.add_edge(START, \"classify\")
graph.add_conditional_edges(\"classify\", route, {
    \"pricing\": \"pricing\",
    \"general\": \"general\",
})
graph.add_edge(\"pricing\", END)
graph.add_edge(\"general\", END)

app = graph.compile()
result = app.invoke({\"query\": \"What is the price?\", \"category\": \"\", \"response\": \"\"})
", Para [Strong [Str "Human-in-the-loop with checkpoint interruption"], Str ":"], CodeBlock ("", ["python"], []) "from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict

class State(TypedDict):
    draft: str
    approved: bool

def generate_draft(state: State) -> dict:
    return {\"draft\": \"Generated content for review.\"}

def finalize(state: State) -> dict:
    return {\"draft\": state[\"draft\"] + \" [APPROVED]\"}

graph = StateGraph(State)
graph.add_node(\"generate\", generate_draft)
graph.add_node(\"finalize\", finalize)

graph.add_edge(START, \"generate\")
graph.add_edge(\"generate\", \"finalize\")
graph.add_edge(\"finalize\", END)

memory = MemorySaver()
app = graph.compile(checkpointer=memory, interrupt_before=[\"finalize\"])

config = {\"configurable\": {\"thread_id\": \"review-1\"}}
result = app.invoke({\"draft\": \"\", \"approved\": False}, config)
# Agent pauses before \"finalize\" for human review
# Resume after approval:
result = app.invoke(None, config)
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Learning curve"], Str ": The graph-based abstraction requires a mental model shift from sequential programming. Debugging conditional edges and state transitions can be non-trivial for complex workflows."]], [Plain [Strong [Str "Ecosystem coupling"], Str ": While LangGraph works standalone, the full production experience (observability, deployment) depends on LangSmith and Agent Server, which are commercial products."]], [Plain [Strong [Str "State serialization"], Str ": All state must be serializable for checkpointing. Complex objects, open connections, or non-serializable resources require careful handling at node boundaries."]], [Plain [Strong [Str "Overhead for simple agents"], Str ": For straightforward single-step or linear agents, the graph abstraction adds unnecessary complexity compared to direct function calls."]], [Plain [Strong [Str "TypeScript parity"], Str ": The TypeScript implementation may lag behind the Python version in feature availability and ecosystem integration."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], Para [Str "LangGraph is under active development as part of the LangChain ecosystem. Key evolutionary milestones include the introduction of durable execution via checkpointing, human-in-the-loop primitives, the comprehensive memory system spanning short-term and long-term persistence, and the Agent Server deployment platform for production workloads."], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "LangGraph Documentation"] ("https://docs.langchain.com/oss/python/langgraph/overview", "")]]]]