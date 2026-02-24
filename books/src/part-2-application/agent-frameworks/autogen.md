# AutoGen

> Microsoft framework for building multi-agent systems with event-driven architecture, supporting collaborative AI workflows through conversational agents, teams, and distributed runtime capabilities.

| Field | Value |
|-------|-------|
| Group | Agent Frameworks |
| Type | SDK |
| Open Source | Yes |
| GitHub | [microsoft/autogen](https://github.com/microsoft/autogen) |
| Stars | 54714 |
| Documentation | [Official Docs](https://microsoft.github.io/autogen/) |

## Overview

AutoGen is an open-source framework developed by Microsoft for building multi-agent systems powered by large language models. It provides an event-driven architecture designed for scalable, distributed agent collaboration. The framework enables developers to create systems where multiple AI agents converse, coordinate, and execute tasks collaboratively. AutoGen requires Python 3.10 or higher and is structured around four integrated layers that span from no-code prototyping to production-grade distributed deployments.

The framework represents a significant evolution from its predecessor (AutoGen 0.2), introducing a redesigned architecture with breaking changes that prioritize event-driven patterns, modular extensibility, and scalable multi-agent orchestration.

## Core Concepts

**Agents** are the fundamental building blocks of AutoGen. Each agent is a conversational entity assigned specific tasks and capabilities. The primary agent type is `AssistantAgent`, which wraps a language model with tool-use abilities and can participate in structured conversations. Agents process messages, generate responses, and can invoke tools to interact with external systems.

**Multi-Agent Systems** are collaborative architectures where multiple agents work together. AutoGen supports three primary patterns: deterministic workflows with predefined agent interaction sequences, dynamic business processes where agents negotiate and delegate based on runtime conditions, and distributed applications where agents run across separate processes or machines.

**Event-Driven Design** underpins the entire framework. Agents communicate through asynchronous events rather than synchronous function calls, enabling horizontal scalability and loose coupling between components. This architecture allows agents to be deployed independently and communicate through message-passing infrastructure.

**Teams** are coordinated groups of agents that collaborate on tasks. A team defines which agents participate, how they communicate, and what termination conditions apply. Teams abstract away the orchestration logic, letting developers focus on individual agent capabilities while the framework handles coordination.

## Installation and Setup

AutoGen provides separate installation paths depending on the layer being used.

For the Studio web-based prototyping interface:

```bash
pip install -U autogenstudio
autogenstudio ui --port 8080 --appdir ./myapp
```

For the AgentChat Python framework with OpenAI model support:

```bash
pip install -U "autogen-agentchat" "autogen-ext[openai]"
```

The modular package structure allows installing only the components needed. The `autogen-agentchat` package provides the high-level agent and team abstractions, while `autogen-ext` contains optional extensions for model providers, code executors, and external service integrations.

## Architecture

AutoGen is organized into four integrated layers, each building on the one below it:

**Studio** is the top layer, providing a web-based interface for prototyping agent workflows without writing code. It allows visual construction and testing of multi-agent systems through a graphical interface served as a local web application.

**AgentChat** is the Python framework layer for building single-agent and multi-agent conversations programmatically. It provides high-level abstractions like `AssistantAgent`, team definitions, and conversation management. This is the primary API surface for most developers.

**Core** is the foundational layer implementing the event-driven architecture. It handles message routing, agent lifecycle management, and runtime orchestration. The Core layer enables scalable and distributed multi-agent deployments through its asynchronous event processing infrastructure.

**Extensions** is the modular integration layer that connects AutoGen to external services and capabilities. Extensions are packaged separately and installed as needed, keeping the core framework lightweight while enabling rich integrations.

## Key Features and Functionality

**Conversational Agent Framework.** Agents communicate through structured conversations with message history, role assignments, and tool invocations. The conversation model supports both single-agent interactions and complex multi-agent dialogues.

**Flexible Orchestration Patterns.** Teams can be configured with different orchestration strategies including round-robin, selector-based routing, and custom termination conditions. This flexibility supports both simple linear workflows and complex branching agent interactions.

**Distributed Runtime Support.** The gRPC-based worker agent runtime enables agents to run in separate processes or on different machines, communicating through network protocols. This supports production deployments that require horizontal scaling.

**Sandboxed Code Execution.** Docker-based code executors allow agents to generate and run code in isolated environments, preventing untrusted code from affecting the host system.

**Model Context Protocol (MCP) Integration.** The McpWorkbench extension enables agents to connect to MCP servers, expanding their tool-use capabilities through the standardized protocol.

**OpenAI Assistant API Integration.** Native support for the OpenAI Assistant API through the `OpenAIAssistantAgent` extension, allowing direct use of OpenAI's managed agent infrastructure within AutoGen workflows.

## Use Cases

**Multi-Agent Collaboration Research.** AutoGen serves as a platform for researching and experimenting with multi-agent communication patterns, negotiation strategies, and emergent collaborative behaviors between AI agents.

**Enterprise Business Automation.** The framework supports building production-grade automation systems where multiple specialized agents handle different aspects of business processes, from data retrieval and analysis to decision-making and action execution.

**Distributed Agentic Workflows.** For applications requiring agents to operate across different services, machines, or geographic locations, AutoGen's event-driven Core layer and gRPC runtime provide the infrastructure for distributed deployment.

**Deterministic Workflow Orchestration.** When predictable, repeatable agent interactions are required, AutoGen's team configurations support fully deterministic execution paths with predefined agent sequences and termination conditions.

## API Reference Summary

The primary entry points for AutoGen development:

**Agents:**
- `AssistantAgent`: Core agent type wrapping a language model with tool-use capabilities. Accepts a name, model client, and optional tool definitions.

**Model Clients:**
- `OpenAIChatCompletionClient`: Client for OpenAI-compatible chat completion APIs. Configured with model name and optional API parameters.

**Teams:**
- Team classes define agent groups with orchestration strategies, selectors, and termination conditions.

**Code Executors:**
- `DockerCommandLineCodeExecutor`: Runs agent-generated code in Docker containers for sandboxed execution.

**Runtime:**
- `GrpcWorkerAgentRuntime`: Distributed runtime enabling agents to communicate across processes via gRPC.

**Extensions:**
- `McpWorkbench`: Connects agents to MCP servers for standardized tool access.
- `OpenAIAssistantAgent`: Wraps the OpenAI Assistant API as an AutoGen agent.

## Configuration and Customization

AutoGen agents are configured programmatically through constructor parameters. A minimal agent configuration requires a name and a model client:

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")
agent = AssistantAgent("assistant", model_client)
```

Model clients accept standard parameters for the underlying API including model name, API key (via environment variables or explicit parameter), temperature, and token limits. Team configurations define agent membership, orchestration strategy, and termination conditions.

The Studio interface stores configurations in a local application directory specified by the `--appdir` flag, enabling persistent workflow definitions across sessions.

## Integration Patterns

**Tool Integration.** Agents can be equipped with Python functions as tools. The framework handles schema generation, invocation, and result parsing. Tools are defined as standard Python functions and registered with agents during construction.

**MCP Server Integration.** Through the McpWorkbench extension, agents connect to any MCP-compatible server, gaining access to its tools, resources, and prompts through the standardized protocol.

**External Model Providers.** The extension system supports multiple model providers beyond OpenAI. Each provider is packaged as a separate extension, allowing teams to use their preferred language model infrastructure.

**Distributed Deployment.** The gRPC worker runtime enables deploying agents as independent services that communicate through network protocols, supporting microservice-style architectures where agents are independently scalable and deployable.

## Examples

Basic single-agent interaction:

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

async def main():
    model_client = OpenAIChatCompletionClient(model="gpt-4o")
    agent = AssistantAgent("assistant", model_client)
    response = await agent.on_messages(
        [{"role": "user", "content": "What is the capital of France?"}],
        cancellation_token=None,
    )
    print(response.chat_message.content)

asyncio.run(main())
```

Studio launch for no-code prototyping:

```bash
pip install -U autogenstudio
autogenstudio ui --port 8080 --appdir ./my_autogen_app
```

## Limitations and Considerations

**Migration Complexity.** AutoGen 0.2 users face breaking changes when upgrading. The redesigned architecture requires rewriting existing agent definitions and workflows to conform to the new event-driven patterns.

**Python Dependency.** The framework requires Python 3.10 or higher, limiting deployment environments. The distributed runtime mitigates this for production but development remains Python-centric.

**Extension Maturity.** As a modular system with separate extension packages, the quality and maintenance status of individual extensions varies. Core extensions maintained by Microsoft are well-supported, while community extensions may have inconsistent update cycles.

**Orchestration Overhead.** Multi-agent systems introduce coordination complexity. Debugging conversation flows across multiple agents, understanding message routing decisions, and diagnosing failures in distributed deployments requires familiarity with the event-driven architecture.

## Changelog Highlights

- Redesigned architecture from AutoGen 0.2 with event-driven Core layer
- Introduction of four-layer architecture: Studio, AgentChat, Core, Extensions
- Addition of gRPC-based distributed agent runtime
- MCP integration through McpWorkbench extension
- Studio web interface for no-code agent prototyping
- Docker-based sandboxed code execution support

## Citations

- [1] AutoGen Documentation - https://microsoft.github.io/autogen/stable/
