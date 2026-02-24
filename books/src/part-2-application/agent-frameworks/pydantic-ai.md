# Pydantic AI

> Python agent framework for building production-grade Generative AI (GenAI) applications with type-safe, structured outputs and a developer experience inspired by FastAPI.

| Field          | Value                                                        |
|----------------|--------------------------------------------------------------|
| Name           | Pydantic AI                                                  |
| Group          | Agent Frameworks                                             |
| Type           | SDK                                                          |
| Open Source    | Yes                                                          |
| GitHub         | [pydantic/pydantic-ai](https://github.com/pydantic/pydantic-ai) |
| Stars          | 15015                                                        |
| Documentation  | [Official Docs](https://ai.pydantic.dev/)                    |
| Python Version | 3.10+                                                        |

## Overview

Pydantic AI is a Python agent framework designed to bring the ergonomics and developer experience of FastAPI to GenAI application and agent development. Built by the creators of Pydantic, it leverages the same validation and serialization engine to provide type-safe, structured outputs from Large Language Model (LLM) interactions. The framework is model-agnostic, supporting a wide range of LLM providers through a unified interface, and emphasizes production readiness through features like dependency injection, observability, and durable execution support.

The core design philosophy centers on treating LLM interactions as typed function calls. Agents are parameterized by dependency and output types, enabling full IDE auto-completion, static type checking, and Pydantic model validation on every response. This approach reduces runtime errors, makes refactoring safer, and brings the same confidence to GenAI development that typed frameworks bring to web development.

## Core Concepts

**Agents** are the primary orchestration unit. An Agent manages the lifecycle of an LLM interaction: sending prompts, handling multi-turn conversations, invoking tools, validating outputs, and retrying on validation failures. Agents are generic, parameterized by a dependency type and an output type, which enables static analysis and IDE support throughout the development workflow.

**Models** represent LLM provider integrations. Pydantic AI supports OpenAI, Anthropic, Google (Gemini and Vertex AI), xAI (Grok), Amazon Bedrock, Mistral, Groq, Cohere, and Hugging Face, among others. The model-agnostic design means application code does not need to change when switching providers. Each model integration handles provider-specific API details, authentication, and message formatting.

**Tools** are Python functions that the LLM can invoke during a conversation. Tools are registered on agents using the `@agent.tool` decorator. The function's docstring automatically becomes the tool description sent to the LLM. Pydantic validates tool arguments before execution; if validation fails, the error is sent back to the LLM for self-correction and retry.

**Dependency Injection** is handled through typed dataclasses passed via `RunContext`. Dependencies provide a type-safe mechanism for supplying external resources (database connections, API clients, configuration) to tools and dynamic instructions without global state. This pattern improves testability by allowing dependencies to be swapped with test doubles.

**Output Types** constrain LLM responses to Pydantic models. The framework generates JSON schemas from the output type and instructs the LLM to conform to the schema. Responses are validated against the model, and validation errors trigger automatic retries with the error details fed back to the LLM.

## Installation and Setup

### Standard Installation

```bash
pip install pydantic-ai
```

Or using uv:

```bash
uv add pydantic-ai
```

### Slim Installation

For minimal dependency footprints, install only the providers needed:

```bash
pip install "pydantic-ai-slim[openai]"
pip install "pydantic-ai-slim[openai,google,logfire]"
```

Available optional groups: `openai`, `anthropic`, `google`, `groq`, `mistral`, `cohere`, `bedrock`, `huggingface`, `vertexai`, `logfire`, `evals`, `cli`, `mcp`, `fastmcp`, `a2a`, `ui`.

### Provider Configuration

Model providers are typically configured through environment variables for API keys (for example, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`) or passed directly when instantiating a model.

## Architecture

Pydantic AI follows a layered architecture:

- **Agent Layer**: Orchestrates the conversation loop, manages tool dispatch, enforces output validation, and handles retries. Agents are the top-level entry point for all LLM interactions.
- **Model Layer**: Abstracts provider-specific APIs behind a common interface. Each provider integration translates between Pydantic AI's internal message format and the provider's API.
- **Tool Layer**: Registered functions with automatic schema generation from type hints and docstrings. Tool execution is sandboxed within the agent's conversation loop.
- **Validation Layer**: Pydantic models define output schemas. The framework validates every LLM response against the declared output type and feeds validation errors back for self-correction.
- **Dependency Layer**: Typed dataclasses injected via `RunContext` provide external resources to tools and dynamic instructions without coupling to global state.

The conversation loop follows a cycle: the agent sends a prompt to the model, the model responds with either a final answer or tool call requests, the agent executes requested tools and sends results back, and this continues until a final validated output is produced or retries are exhausted.

## Key Features and Functionality

1. **Type Safety**: Full IDE auto-completion and static type checking across agents, tools, dependencies, and outputs. Generic type parameters on agents flow through the entire call chain.

2. **Structured Output**: Pydantic model validation on LLM responses. JSON schemas are generated from output types and provided to the LLM, with automatic retry on validation failure.

3. **Dynamic Instructions**: System prompts can be generated at runtime using decorated functions that receive the current `RunContext`, enabling instructions that adapt to the current user, session, or application state.

4. **Observability**: Native integration with Pydantic Logfire for distributed tracing, debugging, and cost monitoring of LLM interactions. Every agent run, tool call, and retry is captured as a span.

5. **Streamed Results**: Continuous structured output streaming with incremental validation. Partial results are available as the LLM generates tokens, with full validation applied on completion.

6. **Model Context Protocol (MCP) Integration**: Connect agents to external tools and data sources via the MCP standard, enabling access to file systems, databases, APIs, and other resources through a standardized protocol.

7. **Human-in-the-Loop**: Tool approval gates allow human review and approval before tool execution, enabling supervised agent workflows where certain actions require explicit confirmation.

8. **Durable Execution**: Integration with workflow engines like Temporal, Database-Backed Operating System (DBOS), and Prefect for long-running agent tasks that need fault tolerance, checkpointing, and resumability.

9. **Graph Support**: Complex multi-step workflows can be modeled as graphs using type hints, enabling branching, conditional logic, and parallel execution paths beyond simple linear agent chains.

10. **Evals Framework**: Built-in systematic performance testing for evaluating agent behavior across test cases, measuring output quality, tool usage patterns, and regression detection.

## Use Cases

- **Structured Data Extraction**: Parsing unstructured text into validated Pydantic models (invoices, resumes, research papers, medical records).
- **Conversational Agents**: Multi-turn chatbots with tool access, maintaining conversation history and enforcing output schemas.
- **Automated Workflows**: Multi-step business processes where each step produces a typed output consumed by the next, with human approval gates at critical decision points.
- **API Integration Agents**: Agents that call external APIs through tools, with dependency injection providing authenticated clients and configuration.
- **Content Generation**: Generating structured content (product descriptions, reports, summaries) with schema-validated outputs ensuring completeness and format compliance.
- **Research and Analysis**: Agents that gather information from multiple sources via tools, synthesize findings, and produce structured analysis reports.

## API Reference Summary

### Agent

- `Agent(model, deps_type, result_type, system_prompt, tools)`: Create an agent with specified model, dependency type, output type, and configuration.
- `agent.run(prompt, deps)`: Execute an agent run asynchronously, returning a validated result.
- `agent.run_sync(prompt, deps)`: Execute an agent run synchronously.
- `agent.run_stream(prompt, deps)`: Execute an agent run with streamed output.
- `@agent.tool`: Decorator to register a tool function on the agent.
- `@agent.system_prompt`: Decorator to register a dynamic system prompt generator.

### RunContext

- `RunContext[DepsType]`: Typed context passed to tools and dynamic instructions, carrying dependencies, retry count, and run metadata.

### Models

- `OpenAIModel`, `AnthropicModel`, `GeminiModel`, `GroqModel`, `MistralModel`, `CohereModel`, `BedrockModel`: Provider-specific model classes.
- `TestModel`: Testing model that returns predictable responses for unit testing.
- `FunctionModel`: Model backed by a custom function for testing and prototyping.

### Results

- `RunResult[OutputType]`: Contains the validated output, message history, cost information, and run metadata.
- `StreamedRunResult[OutputType]`: Streaming variant with incremental access to partial results.

## Configuration and Customization

### Agent Configuration

Agents accept configuration at instantiation and at run time:

- `model`: The LLM model to use (string identifier or model instance).
- `deps_type`: The dependency type for the agent (used for type checking).
- `result_type`: The Pydantic model or type constraining the output.
- `system_prompt`: Static string or list of strings for system instructions.
- `tools`: List of tool functions (alternative to decorator registration).
- `retries`: Maximum number of retries on validation failure (default varies by provider).
- `result_retries`: Maximum retries specifically for output validation failures.

### Model Configuration

Models are configured with provider-specific parameters:

- API keys via environment variables or constructor arguments.
- Base URLs for custom endpoints or proxies.
- Model identifiers (for example, `gpt-4o`, `claude-3-5-sonnet`, `gemini-2.0-flash`).
- Temperature, max tokens, and other generation parameters passed at run time.

## Integration Patterns

### Dependency Injection Pattern

```python
from dataclasses import dataclass
from pydantic_ai import Agent, RunContext

@dataclass
class AppDeps:
    db_connection: DatabaseClient
    api_key: str

agent = Agent('openai:gpt-4o', deps_type=AppDeps)

@agent.tool
async def lookup_user(ctx: RunContext[AppDeps], user_id: int) -> str:
    """Look up user details by ID."""
    return await ctx.deps.db_connection.get_user(user_id)
```

### Structured Output Pattern

```python
from pydantic import BaseModel
from pydantic_ai import Agent

class CityInfo(BaseModel):
    name: str
    country: str
    population: int
    notable_landmarks: list[str]

agent = Agent('openai:gpt-4o', result_type=CityInfo)
result = agent.run_sync('Tell me about Paris')
city: CityInfo = result.data
```

### Dynamic Instructions Pattern

```python
from pydantic_ai import Agent, RunContext

agent = Agent('openai:gpt-4o', deps_type=UserSession)

@agent.system_prompt
async def personalized_prompt(ctx: RunContext[UserSession]) -> str:
    return f"You are assisting {ctx.deps.username}, who prefers {ctx.deps.language}."
```

### Testing Pattern

```python
from pydantic_ai.models.test import TestModel

agent = Agent('openai:gpt-4o', result_type=MyOutput)

with agent.override(model=TestModel()):
    result = agent.run_sync('test input')
    assert isinstance(result.data, MyOutput)
```

## Examples

### Basic Agent with Tools

```python
from pydantic_ai import Agent

agent = Agent(
    'openai:gpt-4o',
    system_prompt='You are a helpful assistant that can check the weather.',
)

@agent.tool
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    return f"The weather in {city} is sunny, 22 degrees Celsius."

result = agent.run_sync('What is the weather in London?')
print(result.data)
```

### Structured Data Extraction

```python
from pydantic import BaseModel
from pydantic_ai import Agent

class Invoice(BaseModel):
    vendor: str
    total: float
    currency: str
    line_items: list[str]

agent = Agent('openai:gpt-4o', result_type=Invoice)
result = agent.run_sync('Extract: Invoice from Acme Corp, $150.00 USD for widgets and gizmos')
invoice: Invoice = result.data
```

### Multi-Turn Conversation

```python
from pydantic_ai import Agent

agent = Agent('openai:gpt-4o', system_prompt='You are a math tutor.')

result1 = agent.run_sync('What is the derivative of x squared?')
result2 = agent.run_sync(
    'What about x cubed?',
    message_history=result1.new_messages(),
)
print(result2.data)
```

## Limitations and Considerations

- **Python Only**: No support for other programming languages. Requires Python 3.10 or higher.
- **Synchronous Overhead**: While async is fully supported, the synchronous `run_sync` method runs an event loop internally, which may conflict with existing event loops in some environments.
- **Provider Feature Parity**: Not all LLM providers support all features equally. Structured output support, tool calling capabilities, and streaming behavior vary by provider.
- **Validation Retry Cost**: Automatic retries on validation failure consume additional tokens and increase latency. Poorly specified output types can lead to retry loops.
- **Graph Complexity**: The graph-based workflow system, while powerful, adds complexity compared to simpler linear agent patterns and may require careful design for non-trivial workflows.
- **Ecosystem Maturity**: As a relatively new framework (first released in late 2024), the ecosystem of community plugins, tutorials, and third-party integrations is still growing.

## Changelog Highlights

- **Graph Support**: Added typed graph-based workflows for complex multi-step agent orchestration.
- **MCP Integration**: Native support for the Model Context Protocol, enabling standardized external tool and data access.
- **Agent-to-Agent (A2A) Protocol**: Support for inter-agent communication via the A2A standard.
- **Human-in-the-Loop**: Tool approval gates for supervised agent execution.
- **Evals Framework**: Built-in systematic testing for evaluating agent performance and detecting regressions.
- **Expanded Provider Support**: Ongoing additions including Bedrock, Cohere, Hugging Face, and Vertex AI integrations.
- **UI Integration**: Built-in support for agent user interfaces.
- **Durable Execution**: Partnerships with Temporal, DBOS, and Prefect for fault-tolerant long-running agents.

## Citations

- [1] Pydantic AI Documentation - https://ai.pydantic.dev/
- [2] Pydantic AI Installation - https://ai.pydantic.dev/install/
