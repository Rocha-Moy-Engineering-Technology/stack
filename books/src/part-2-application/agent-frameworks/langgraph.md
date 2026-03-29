# LangGraph

> Low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents with durable execution, human-in-the-loop capabilities, and comprehensive memory support.

| Field | Value |
|-------|-------|
| Group | Agent Frameworks |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) |
| Stars | 24935 |
| Documentation | [Official Docs](https://docs.langchain.com/oss/python/langgraph/overview) |

## Overview

LangGraph is a low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents. Part of the LangChain ecosystem, it provides graph-based workflow composition where agents are defined as nodes connected by edges, with shared state flowing through the execution graph. LangGraph is available in both Python and TypeScript, and it operates standalone or alongside LangChain and LangSmith for observability and evaluation.

Unlike higher-level agent frameworks that prescribe specific LLM patterns or tool-calling approaches, LangGraph provides the primitives for constructing arbitrary agent architectures. Developers define computational nodes, wire them together with conditional or direct edges, and let the framework handle state management, checkpointing, and durable execution. This low-level design makes LangGraph suitable for complex decision logic, extended execution tasks, failure recovery, and multi-agent systems that require fine-grained control over agent behavior.

## Core Concepts

**Graphs** are the foundational abstraction in LangGraph. An agent workflow is represented as a directed graph where nodes perform computation and edges define transitions between them. The graph structure determines how state flows through the system and which nodes execute in which order.

**Nodes** are the computational units within a graph. Each node is a function that receives the current state, performs some operation (such as calling an LLM, executing a tool, or transforming data), and returns an updated state. Nodes encapsulate discrete steps of agent logic.

**Edges** define transitions between nodes. They can be direct (always transition from node A to node B) or conditional (evaluate a predicate on the current state to determine which node executes next). Conditional edges enable branching logic, loops, and dynamic routing within agent workflows.

**State** is a shared data structure that flows through the graph. It accumulates information as nodes process it, serving as the working context for the entire agent execution. State is typed and structured, providing a predictable contract between nodes.

**Channels** enable stateful data flow within the graph. They define how state updates are merged and propagated, supporting patterns like message accumulation, key-value storage, and append-only logs.

**Checkpointing** provides durable execution by persisting agent state at defined points. If an agent fails or is interrupted, execution resumes from the most recent checkpoint rather than restarting from scratch. This is critical for long-running workflows that interact with external systems.

**Human-in-the-Loop** support allows developers to inspect and modify agent state at any point during execution. Agents can pause, present their current state for human review or correction, and then resume with the modified state. This enables supervised agent deployments where human oversight is required.

**Memory** in LangGraph spans two dimensions: short-term working memory within a single session and persistent long-term memory that carries across sessions. This dual-memory model allows agents to maintain context within a conversation while also recalling information from previous interactions.

## Architecture

LangGraph follows a graph-based orchestration architecture with no mandated LLM patterns or tool-calling approaches. The core runtime executes a directed graph where:

1. **State initialization** creates the shared data structure that flows through all nodes
2. **Node execution** processes state through computational units in topological order, respecting edge conditions
3. **Edge evaluation** determines the next node(s) to execute based on direct connections or conditional predicates
4. **Checkpoint persistence** saves state at configurable points for durability
5. **Completion** occurs when execution reaches a terminal node or an explicit end condition

The architecture separates the graph definition (compile-time) from graph execution (runtime). Graphs are defined declaratively, compiled into an executable form, and then invoked with initial state. This separation allows the same graph definition to be executed with different checkpointing backends, memory configurations, and deployment targets.

LangGraph operates independently but integrates with the broader LangChain ecosystem:

- **Standalone**: Pure graph-based orchestration without LangChain dependencies
- **With LangChain**: Access to pre-built agent abstractions, tool integrations, and prompt management
- **With LangSmith**: Observability, tracing, evaluation, and monitoring of agent executions
- **With Agent Server**: Deployment platform for stateful agent workflows with scaling infrastructure

## Key Features and Functionality

**Durable Execution** ensures agents persist through failures and infrastructure disruptions. By checkpointing state at defined intervals, LangGraph enables agents to resume from the last successful point rather than restarting entirely. This is essential for workflows that span minutes, hours, or days.

**Human-in-the-Loop Control** provides mechanisms to pause agent execution, surface the current state for human inspection or modification, and resume with updated state. This supports use cases where regulatory compliance, safety constraints, or domain expertise require human oversight at critical decision points.

**Comprehensive Memory System** offers both short-term working memory within a session and persistent long-term memory across sessions. Agents can maintain conversational context, remember user preferences, and accumulate knowledge over time without external memory infrastructure.

**Conditional Routing** through conditional edges enables dynamic agent behavior. Based on the current state, agents can branch into different execution paths, retry failed operations, or loop through iterative refinement steps. This goes beyond linear pipelines to support genuinely adaptive agent logic.

**Multi-Agent Composition** allows multiple agent graphs to be composed into larger systems. Individual agents can be nodes within a parent graph, enabling hierarchical agent architectures where supervisor agents delegate to specialized sub-agents.

**Production Deployment** via Agent Server provides scalable infrastructure for running stateful agent workflows. This includes horizontal scaling, persistent state management, and operational tooling for monitoring agent health and performance.

## Use Cases

- **Extended execution tasks**: Multi-step research, data processing pipelines, and workflows that run for extended periods and must survive infrastructure disruptions
- **Complex decision logic**: Agents that require branching, looping, and conditional routing based on intermediate results
- **Human oversight workflows**: Regulated industries or high-stakes decisions where human review is required before agent actions are finalized
- **Failure recovery**: Long-running processes that interact with unreliable external systems and need checkpoint-based recovery
- **Multi-agent systems**: Architectures where multiple specialized agents collaborate, with a supervisor agent orchestrating their interactions
- **Conversational agents with memory**: Chatbots and assistants that maintain context within and across sessions

## API Reference Summary

**Graph Construction**:

- `StateGraph(state_schema)` -- Create a new graph with a typed state schema
- `graph.add_node(name, function)` -- Add a computational node to the graph
- `graph.add_edge(source, target)` -- Add a direct edge between nodes
- `graph.add_conditional_edges(source, condition_fn, mapping)` -- Add conditional routing from a node
- `graph.compile(checkpointer=None)` -- Compile the graph into an executable application

**Execution**:

- `app.invoke(initial_state, config)` -- Execute the graph synchronously with initial state
- `app.ainvoke(initial_state, config)` -- Execute the graph asynchronously
- `app.stream(initial_state, config)` -- Stream execution results as they are produced

**Built-in References**:

- `START` -- Special node representing the graph entry point
- `END` -- Special node representing the graph terminal point

**Checkpointing**:

- `MemorySaver()` -- In-memory checkpoint storage for development
- `SqliteSaver(connection)` -- SQLite-backed checkpoint storage
- `PostgresSaver(connection)` -- PostgreSQL-backed checkpoint storage

**Configuration**:

- `thread_id` -- Identifies a conversation or execution thread for checkpointing
- `checkpoint_id` -- References a specific checkpoint for resumption

## Configuration and Customization

LangGraph configuration is primarily code-driven through the graph compilation step and invocation config:

```python
# Checkpointer configuration
from langgraph.checkpoint.memory import MemorySaver
checkpointer = MemorySaver()
app = graph.compile(checkpointer=checkpointer)

# Thread configuration for state persistence
config = {
    "configurable": {
        "thread_id": "user-session-123",
    }
}

# Invoke with configuration
result = app.invoke(initial_state, config)
```

For production deployments, Agent Server configuration handles scaling, persistence backends, and operational parameters separately from the graph definition.

## Integration Patterns

**LangChain Integration**: LangGraph works with LangChain's chat models, tools, and prompt templates. LangChain provides higher-level abstractions that can be used within LangGraph nodes:

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph

llm = ChatOpenAI(model="gpt-4")

def call_model(state):
    response = llm.invoke(state["messages"])
    return {"messages": state["messages"] + [response]}
```

**LangSmith Observability**: When LangSmith is configured, LangGraph executions are automatically traced, providing visibility into node-level execution times, state transitions, and LLM calls.

**Tool Integration**: LangGraph nodes can invoke any callable, including LangChain tools, custom functions, API clients, and database operations. There is no restriction on what a node can execute.

**External State Stores**: Checkpointing backends can be swapped between in-memory (development), SQLite (single-node), and PostgreSQL (production) without changing graph logic.

## Examples

**Conditional routing based on agent state**:

```python
from langgraph.graph import StateGraph, START, END
from typing import TypedDict, Literal

class State(TypedDict):
    query: str
    category: str
    response: str

def classify(state: State) -> dict:
    if "price" in state["query"].lower():
        return {"category": "pricing"}
    return {"category": "general"}

def handle_pricing(state: State) -> dict:
    return {"response": "Routing to pricing information."}

def handle_general(state: State) -> dict:
    return {"response": "Routing to general support."}

def route(state: State) -> Literal["pricing", "general"]:
    return state["category"]

graph = StateGraph(State)
graph.add_node("classify", classify)
graph.add_node("pricing", handle_pricing)
graph.add_node("general", handle_general)

graph.add_edge(START, "classify")
graph.add_conditional_edges("classify", route, {
    "pricing": "pricing",
    "general": "general",
})
graph.add_edge("pricing", END)
graph.add_edge("general", END)

app = graph.compile()
result = app.invoke({"query": "What is the price?", "category": "", "response": ""})
```

**Human-in-the-loop with checkpoint interruption**:

```python
from langgraph.graph import StateGraph, START, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict

class State(TypedDict):
    draft: str
    approved: bool

def generate_draft(state: State) -> dict:
    return {"draft": "Generated content for review."}

def finalize(state: State) -> dict:
    return {"draft": state["draft"] + " [APPROVED]"}

graph = StateGraph(State)
graph.add_node("generate", generate_draft)
graph.add_node("finalize", finalize)

graph.add_edge(START, "generate")
graph.add_edge("generate", "finalize")
graph.add_edge("finalize", END)

memory = MemorySaver()
app = graph.compile(checkpointer=memory, interrupt_before=["finalize"])

config = {"configurable": {"thread_id": "review-1"}}
result = app.invoke({"draft": "", "approved": False}, config)
# Agent pauses before "finalize" for human review
# Resume after approval:
result = app.invoke(None, config)
```

## Limitations and Considerations

- **Learning curve**: The graph-based abstraction requires a mental model shift from sequential programming. Debugging conditional edges and state transitions can be non-trivial for complex workflows.
- **Ecosystem coupling**: While LangGraph works standalone, the full production experience (observability, deployment) depends on LangSmith and Agent Server, which are commercial products.
- **State serialization**: All state must be serializable for checkpointing. Complex objects, open connections, or non-serializable resources require careful handling at node boundaries.
- **Overhead for simple agents**: For straightforward single-step or linear agents, the graph abstraction adds unnecessary complexity compared to direct function calls.
- **TypeScript parity**: The TypeScript implementation may lag behind the Python version in feature availability and ecosystem integration.

## Changelog Highlights

LangGraph is under active development as part of the LangChain ecosystem. Key evolutionary milestones include the introduction of durable execution via checkpointing, human-in-the-loop primitives, the comprehensive memory system spanning short-term and long-term persistence, and the Agent Server deployment platform for production workloads.

## Citations

- [1] [LangGraph Documentation](https://docs.langchain.com/oss/python/langgraph/overview)
