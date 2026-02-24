# ADK

> Google Agent Development Kit -- a modular, open-source framework for developing and deploying AI agents, optimized for the Gemini and Google ecosystem while remaining model-agnostic and deployment-agnostic.

| Field         | Value                                                        |
|---------------|--------------------------------------------------------------|
| Name          | ADK                                                          |
| Group         | Agent Frameworks                                             |
| Type          | SDK                                                          |
| Open Source   | Yes                                                          |
| GitHub        | [google/adk-python](https://github.com/google/adk-python)   |
| Stars         | 17911                                                        |
| Documentation | [Official Docs](https://google.github.io/adk-docs/)         |

## Overview

Google Agent Development Kit (ADK) is a framework that makes "agent development feel more like software development." It provides a code-first, modular approach to building, evaluating, and deploying AI agents. ADK is optimized for the Gemini model family and the broader Google ecosystem (Vertex AI, Google Cloud) but is designed to be model-agnostic and deployment-agnostic, allowing developers to integrate models from various providers and run agents in diverse environments.

ADK supports multiple programming languages -- Python, TypeScript, Go, and Java -- making it accessible across different development teams and technology stacks. The framework emphasizes composability through its multi-agent architecture, where specialized agents can be organized hierarchically and orchestrated through both structured workflows and dynamic LLM-driven routing.

## Core Concepts

**Agents** are the fundamental building blocks in ADK. There are three primary categories:

- **LLM Agents**: Driven by a language model, these agents use an LLM to reason, plan, and decide which tools to invoke or which sub-agents to delegate to. The core implementation is `LlmAgent` (aliased as `Agent`), which accepts a model, instructions, tools, and optional sub-agents.
- **Workflow Agents**: Provide deterministic, structured orchestration without relying on an LLM for control flow. Includes `SequentialAgent` (executes sub-agents in order), `ParallelAgent` (executes sub-agents concurrently), and `LoopAgent` (repeats sub-agents until an exit condition is met).
- **Custom Agents**: Extend `BaseAgent` to implement arbitrary orchestration logic, giving developers full control over agent behavior.

**Tools** extend what agents can do beyond text generation. ADK supports function tools (plain Python/TypeScript functions), built-in tools (Google Search, code execution), third-party integrations (MCP tools, OpenAPI-based tools, LangChain tools), and the agent-as-tool pattern where one agent can be used as a tool by another.

**Sessions and State** manage conversation context and agent memory. A session tracks the interaction history between a user and an agent, while state provides a key-value store for persisting information across turns. ADK includes support for in-memory, database-backed, and Vertex AI-managed session services.

**Artifacts** provide a mechanism for agents to store and retrieve files or binary data (images, documents, generated outputs) associated with a session.

**Callbacks** allow developers to hook into agent and tool execution at various lifecycle points (before/after agent calls, before/after tool calls, before/after model calls), enabling logging, guardrails, and custom modifications to behavior.

## Installation and Setup

### Python

```bash
pip install google-adk
```

### TypeScript

```bash
npm install @google/adk
```

### Go

```bash
go get google.golang.org/adk
```

### Java

Available via Maven or Gradle. Add the ADK dependency to your build configuration following the official documentation for the latest coordinates.

### Quick Start (Python)

A minimal agent definition consists of creating an agent with a model, name, and instructions:

```python
from google.adk.agents import Agent

root_agent = Agent(
    name="greeting_agent",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant. Greet the user warmly.",
)
```

To run the agent locally with the built-in web interface:

```bash
adk web
```

Or via the command line:

```bash
adk run <agent_directory>
```

### Environment Configuration

ADK requires API keys or credentials depending on the model provider:

- **Gemini (Google AI Studio)**: Set the `GOOGLE_API_KEY` environment variable.
- **Vertex AI**: Set `GOOGLE_CLOUD_PROJECT` and `GOOGLE_CLOUD_LOCATION`, and ensure Application Default Credentials are configured.

## Architecture

ADK follows a layered, modular architecture designed around composability:

**Agent Layer**: The topmost layer where agent definitions live. Agents are organized in a tree structure with a single root agent that can delegate to sub-agents. Each agent has its own model configuration, instructions, tools, and optional sub-agents.

**Runtime Layer**: Manages the execution lifecycle of agents. The `Runner` class orchestrates the flow: it receives user input, invokes the appropriate agent, manages tool calls, handles sub-agent delegation, and yields events back to the caller. The runtime supports both synchronous and asynchronous execution, as well as streaming.

**Session and Memory Layer**: Provides state persistence across interactions. `SessionService` implementations manage session creation, retrieval, and storage. Memory services allow agents to access long-term information beyond the current session context.

**Tool Layer**: Abstracts tool execution. Tools are registered with agents and invoked automatically by the LLM when needed. The framework handles serialization of tool inputs/outputs and supports both synchronous and asynchronous tool execution.

**Model Layer**: Abstracts the LLM provider. ADK ships with built-in support for Gemini (via Google AI Studio and Vertex AI) and provides a `BaseLlm` interface for integrating other model providers such as Claude (via Anthropic API or Vertex), Ollama, vLLM, and LiteLLM.

**Deployment Layer**: Agents can be deployed locally (CLI, web UI), as API servers, on Vertex AI Agent Engine, on Cloud Run, or in Docker containers.

## Key Features

**Flexible Orchestration**: Combine structured workflow agents (Sequential, Parallel, Loop) with dynamic LLM-driven routing to build complex agent behaviors. Workflow agents provide predictable, repeatable execution paths while LLM agents handle open-ended reasoning and adaptive decision-making.

**Multi-Agent Architecture**: Build modular applications by composing specialized agents into a hierarchy. Each agent handles a focused domain, and the framework manages delegation and context transfer between agents. Agents can be developed, tested, and iterated on independently.

**Rich Tool Ecosystem**: Integrate with a wide variety of tools out of the box. Function tools let developers wrap any function as a tool with automatic schema generation. MCP (Model Context Protocol) tool support enables compatibility with the growing MCP ecosystem. OpenAPI tools allow agents to interact with any REST API defined by an OpenAPI specification.

**Agent-as-Tool**: Use an entire agent (with its own tools and sub-agents) as a tool for another agent. This enables reuse of complex agent behaviors as building blocks without transferring control flow.

**Built-in Evaluation**: Assess agent quality against predefined test cases. Evaluate final responses, tool usage patterns, and intermediate trajectory steps. Supports custom evaluators and integration with continuous testing pipelines.

**Streaming and Bidirectional Communication**: Support for server-sent events and bidirectional streaming enables real-time, interactive agent experiences, including audio and video streaming for multimodal interactions.

**Guardrails and Callbacks**: Implement safety checks, input validation, output filtering, and custom logic at multiple points in the agent execution lifecycle through the callback system.

**Session and State Management**: Persist conversation history and arbitrary state across turns and sessions. Choose from in-memory storage for development, database-backed storage for production, or managed storage on Vertex AI.

## Use Cases

- **Customer Service Agents**: Multi-agent systems where specialized agents handle different domains (billing, technical support, account management) with a routing agent directing queries to the appropriate specialist.
- **Data Analysis Pipelines**: Sequential agents that retrieve data, transform it, perform analysis, and generate reports, with each step handled by a dedicated agent.
- **Research Assistants**: Agents equipped with search tools, document retrieval, and code execution capabilities to help users explore topics, summarize findings, and generate insights.
- **Workflow Automation**: Orchestrating multi-step business processes with parallel execution where possible and sequential execution where dependencies exist.
- **Conversational Interfaces**: Building chat-based applications with persistent memory, tool access, and the ability to delegate to specialized sub-agents.
- **Code Generation and Review**: Agents that write, test, and review code using code execution tools and structured validation workflows.

## API Reference Summary

### Core Agent Classes

- `Agent` (alias for `LlmAgent`): Primary LLM-driven agent with model, instruction, tools, and sub-agents.
- `SequentialAgent`: Executes sub-agents in defined order.
- `ParallelAgent`: Executes sub-agents concurrently.
- `LoopAgent`: Repeats sub-agents until an escalation or exit condition.
- `BaseAgent`: Abstract base class for building custom agent types.

### Runner

- `Runner`: Orchestrates agent execution; accepts `agent`, `app_name`, and `session_service`. Primary method is `run_async()` which yields `Event` objects.
- `InMemoryRunner`: Convenience runner with built-in in-memory session management for quick prototyping.

### Tools

- `FunctionTool`: Wraps a Python/TypeScript function as a tool with automatic schema inference.
- `google_search`: Built-in Google Search tool.
- `code_execution`: Built-in code execution sandbox tool.
- `MCPToolset.from_server()`: Load tools from an MCP-compliant server.

### Sessions

- `InMemorySessionService`: Development-oriented session storage.
- `DatabaseSessionService`: Production session storage backed by a database.
- `VertexAiSessionService`: Managed session storage on Vertex AI.

### Callbacks

- `before_agent_callback` / `after_agent_callback`: Hook into agent invocation lifecycle.
- `before_tool_callback` / `after_tool_callback`: Hook into tool execution lifecycle.
- `before_model_callback` / `after_model_callback`: Hook into model call lifecycle.

### CLI

- `adk run <agent_dir>`: Run an agent from the command line.
- `adk web`: Launch the built-in web development UI.
- `adk api_server`: Start an API server for the agent.
- `adk eval`: Run evaluation test cases against an agent.
- `adk deploy`: Deploy an agent to Cloud Run or Vertex AI.

## Configuration

### Agent Configuration

Agents are configured programmatically through constructor parameters:

```python
agent = Agent(
    name="my_agent",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant.",
    tools=[my_tool_function],
    sub_agents=[specialist_agent],
    output_key="result",        # Store output in session state
    generate_content_config=GenerateContentConfig(
        temperature=0.7,
        max_output_tokens=1024,
    ),
)
```

### Model Configuration

Specify the model as a string identifier or configure a custom LLM wrapper:

- `"gemini-2.0-flash"`: Google AI Studio Gemini model.
- `"vertexai/gemini-2.0-flash"`: Vertex AI-hosted Gemini model.
- Custom `BaseLlm` subclass: For third-party model providers.

### Environment Variables

- `GOOGLE_API_KEY`: API key for Google AI Studio.
- `GOOGLE_CLOUD_PROJECT`: GCP project ID for Vertex AI.
- `GOOGLE_CLOUD_LOCATION`: GCP region for Vertex AI.
- `ANTHROPIC_API_KEY`: API key when using Claude models via LiteLLM integration.

## Integration Patterns

### MCP (Model Context Protocol) Integration

ADK can consume tools from any MCP-compliant server, enabling interoperability with the broader MCP tool ecosystem:

```python
from google.adk.tools.mcp_tool import MCPToolset

tools, cleanup = await MCPToolset.from_server(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/path"],
    )
)
```

### OpenAPI Tool Integration

Generate tools automatically from an OpenAPI specification:

```python
from google.adk.tools.openapi_tool import OpenAPIToolset

toolset = OpenAPIToolset(
    spec_str=open("openapi.yaml").read(),
    spec_str_type="yaml",
)
```

### LangChain Tool Integration

Wrap existing LangChain tools for use within ADK agents, enabling reuse of the LangChain tool ecosystem.

### Multi-Agent Delegation

Configure parent-child relationships between agents for automatic delegation:

```python
root_agent = Agent(
    name="router",
    model="gemini-2.0-flash",
    instruction="Route user queries to the appropriate specialist.",
    sub_agents=[billing_agent, support_agent, sales_agent],
)
```

The LLM-driven root agent decides which sub-agent to delegate to based on the user query and agent descriptions.

### Vertex AI Deployment

Deploy agents to Vertex AI Agent Engine for managed, scalable hosting:

```bash
adk deploy --project=my-project --region=us-central1
```

## Examples

### Basic Tool-Using Agent

```python
from google.adk.agents import Agent

def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"The weather in {city} is sunny, 22 degrees Celsius."

weather_agent = Agent(
    name="weather_agent",
    model="gemini-2.0-flash",
    instruction="Help users check the weather. Use the get_weather tool.",
    tools=[get_weather],
)
```

### Sequential Workflow

```python
from google.adk.agents import SequentialAgent, Agent

researcher = Agent(
    name="researcher",
    model="gemini-2.0-flash",
    instruction="Research the given topic and store findings in state.",
    output_key="research_findings",
)

writer = Agent(
    name="writer",
    model="gemini-2.0-flash",
    instruction="Write a summary based on the research findings in state.",
)

pipeline = SequentialAgent(
    name="research_pipeline",
    sub_agents=[researcher, writer],
)
```

### Multi-Agent with Routing

```python
from google.adk.agents import Agent

billing_agent = Agent(
    name="billing",
    model="gemini-2.0-flash",
    instruction="Handle billing inquiries. Check balances and process payments.",
    tools=[check_balance, process_payment],
)

support_agent = Agent(
    name="support",
    model="gemini-2.0-flash",
    instruction="Handle technical support issues. Troubleshoot and escalate.",
    tools=[lookup_issue, create_ticket],
)

router = Agent(
    name="customer_service",
    model="gemini-2.0-flash",
    instruction="Route customer queries to billing or support based on intent.",
    sub_agents=[billing_agent, support_agent],
)
```

## Limitations

- **Gemini Optimization**: While model-agnostic in design, ADK is most thoroughly tested and optimized for Gemini models. Non-Gemini models may require additional configuration and may not support all features (such as native streaming or certain tool calling conventions).
- **Evolving API Surface**: As a relatively new and actively developed framework, the API surface may change between releases. Breaking changes are possible in minor versions during the early lifecycle.
- **Language Parity**: The Python SDK is the most mature. TypeScript, Go, and Java SDKs may lag behind in feature completeness.
- **Debugging Complexity**: Multi-agent hierarchies with dynamic routing can be difficult to debug and trace, especially when agents delegate across multiple levels.
- **State Management Scope**: Session state is scoped to a single agent tree and session. Cross-session and cross-agent-tree state sharing requires external mechanisms.
- **Evaluation Tooling**: Built-in evaluation focuses on response quality and tool usage correctness. More complex evaluation scenarios (multi-turn coherence, long-horizon task completion) may require custom evaluation logic.

## Changelog Highlights

ADK is under active development with frequent releases. Key milestones include:

- **Initial Release**: Open-sourced with Python SDK, supporting Gemini models, multi-agent orchestration, tool integration, and local development tooling.
- **Multi-Language Support**: Added TypeScript, Go, and Java SDKs to broaden accessibility.
- **MCP Integration**: Added native support for consuming tools from MCP-compliant servers.
- **Streaming Enhancements**: Introduced bidirectional streaming for real-time audio and video interactions.
- **Deployment Options**: Expanded deployment targets to include Vertex AI Agent Engine, Cloud Run, and Docker.

Refer to the [official documentation](https://google.github.io/adk-docs/) and the [GitHub releases](https://github.com/google/adk-python/releases) for the complete changelog.

## Citations

- [1] [ADK Documentation](https://google.github.io/adk-docs/)
