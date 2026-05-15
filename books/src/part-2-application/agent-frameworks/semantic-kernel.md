# Semantic Kernel

> Enterprise-grade, lightweight SDK for integrating Large Language Models (LLMs) into conventional applications across C#, Python, and Java, providing modular plugin architecture and AI service orchestration.

| Field | Value |
|-------|-------|
| Group | Agent Frameworks |
| Type | SDK |
| Open Source | Yes |
| GitHub | [microsoft/semantic-kernel](https://github.com/microsoft/semantic-kernel) |
| Stars | 27904 |
| Documentation | [Official Docs](https://learn.microsoft.com/en-us/semantic-kernel/) |

## Overview

Semantic Kernel is a lightweight, open-source SDK developed by Microsoft for integrating LLMs into C#, Python, and Java applications. It serves as enterprise-grade middleware for building AI agents, providing a structured approach to combining conventional code with AI model capabilities. The SDK reached version 1.0+ with a commitment to non-breaking changes, making it suitable for production deployments in regulated industries. Semantic Kernel is used internally by Microsoft and adopted by Fortune 500 companies, with compliance support for HIPAA, SOC 2, and GDPR.

The SDK follows a plugin-based architecture where developers expose existing code as functions that AI models can discover and invoke. This design allows teams to incrementally add AI capabilities to existing applications without rewriting business logic. Semantic Kernel handles prompt construction, AI service routing, response parsing, and function orchestration through a central kernel object that acts as both a Dependency Injection (DI) container and an execution pipeline.

## Core Concepts

**Kernel.** The central orchestration object that manages all services and plugins. The kernel selects the appropriate AI service for a given request, builds prompts from templates and context, sends them to the AI model, and parses the responses. It supports events and middleware at each step of this pipeline, enabling cross-cutting concerns such as logging, telemetry, and content filtering. The kernel is modeled after the .NET Service Provider pattern, making it familiar to enterprise .NET developers.

**Services.** AI services (such as chat completion models) and infrastructure services (logging, HTTP clients, telemetry) registered with the kernel. Services are resolved through the kernel's DI container, allowing runtime selection of AI providers based on request characteristics. Multiple AI services can be registered simultaneously, with the kernel choosing the appropriate one per invocation.

**Plugins.** Components that AI services use to perform work in the real world. Plugins retrieve data, call APIs, manipulate files, and execute arbitrary business logic. Each plugin contains one or more kernel functions that are described with metadata so the AI model can understand when and how to invoke them. Plugins bridge the gap between the AI model's reasoning and the application's capabilities.

**Functions.** The individual callable units within plugins. Functions can be native code decorated with `@kernel_function` (Python) or equivalent attributes in C# and Java, or they can be prompt templates that the kernel renders and sends to an AI service. Native functions execute conventional code, while prompt functions generate AI completions. Both types are treated uniformly by the kernel's function-calling infrastructure.

**AI Connectors.** Integrations with AI model providers including OpenAI, Azure OpenAI, Google, and Anthropic. Connectors abstract the provider-specific API details behind a common interface, allowing developers to swap models without rewriting application code. Each connector handles authentication, request formatting, and response deserialization for its target provider.

**Model Context Protocol (MCP) Server.** A recent addition that allows developers to create MCP servers directly from kernel functions, exposing them over Server-Sent Events (SSE) and stdio transports. Prompt templates can also be exposed as MCP prompts, enabling interoperability with the broader MCP ecosystem.

## Architecture

Semantic Kernel follows a layered pipeline architecture centered on the kernel object:

1. **Request Layer.** The application submits a prompt or function call request to the kernel.
2. **Service Selection.** The kernel evaluates registered AI services and selects the appropriate one based on service selectors, model capabilities, or explicit configuration.
3. **Prompt Rendering.** If the request involves a prompt template, the kernel renders it by injecting variables, function results, and context from the conversation history.
4. **Pre-Invocation Filters.** Registered filters and hooks execute before the AI service call, enabling prompt modification, content safety checks, or request logging.
5. **AI Service Invocation.** The selected AI connector sends the rendered prompt to the model provider and receives the response.
6. **Function Calling Loop.** If the AI response includes function call requests, the kernel resolves the target plugin functions, executes them, appends the results to the conversation, and re-invokes the AI service. This loop continues until the model produces a final response without further function calls.
7. **Post-Invocation Filters.** Registered filters execute after the AI response, enabling response modification, telemetry recording, or content filtering.
8. **Response Layer.** The processed response is returned to the application.

The plugin system is flat by design. Plugins register with the kernel at startup, and their function metadata (descriptions, parameter types, return types) is automatically serialized into the AI model's function-calling schema. The kernel manages the serialization and deserialization of function arguments and return values transparently.

## Key Features and Functionality

- **Multi-language support.** First-class SDKs for C#, Python, and Java with consistent abstractions and API surface across all three languages.
- **Plugin architecture.** Expose existing code as AI-callable functions through simple decorators or attributes, without modifying the underlying business logic.
- **OpenAPI specification support.** Import OpenAPI specs as plugins, making any REST API available to the AI model. Compatible with Microsoft 365 Copilot plugin format.
- **Model portability.** Swap between OpenAI, Azure OpenAI, Google, and Anthropic models by changing the AI connector configuration without altering application code.
- **Telemetry and observability.** Built-in support for OpenTelemetry-compatible instrumentation, providing visibility into prompt rendering, AI service calls, and function execution.
- **Hooks and filters.** Pre- and post-invocation filters at both the prompt and function levels, enabling content moderation, logging, retry logic, and custom middleware.
- **MCP server creation.** Generate MCP-compliant servers from kernel functions, exposing them over SSE and stdio transports for interoperability with MCP clients.
- **Prompt templates as MCP prompts.** Semantic Kernel prompt templates can be published as MCP prompts, enabling external tools to discover and invoke them.
- **Dependency injection integration.** Native DI support in C# through the standard `IServiceProvider` pattern, and analogous service registration in Python and Java.
- **Non-breaking changes commitment.** Version 1.0+ maintains backward compatibility, providing stability for production deployments.

## Use Cases

- **Enterprise AI assistants.** Building internal copilots that combine LLM reasoning with access to corporate APIs, databases, and document repositories through plugins.
- **Customer service automation.** Orchestrating multi-step workflows where the AI agent queries knowledge bases, retrieves order information, and executes actions through function calling.
- **Document processing pipelines.** Combining AI-powered extraction and summarization with conventional data validation and storage logic.
- **Multi-model orchestration.** Routing different types of requests to specialized models (such as using a smaller model for classification and a larger model for generation) through the service selection mechanism.
- **Legacy system AI integration.** Wrapping existing REST APIs and code libraries as plugins to make them accessible to AI agents without rewriting the underlying systems.
- **Compliance-sensitive deployments.** Leveraging content filters and audit hooks for regulated industries that require logging and content moderation of all AI interactions.

## API Reference Summary

### Kernel

- `Kernel()` -- Create a new kernel instance (Python).
- `Kernel.CreateBuilder()` -- Create a kernel builder (C#).
- `kernel.add_service(service)` -- Register an AI or infrastructure service.
- `kernel.add_plugin(plugin, plugin_name)` -- Register a plugin with the kernel.
- `kernel.invoke(function, arguments)` -- Execute a kernel function with the given arguments.
- `kernel.invoke_prompt(prompt, arguments)` -- Render and execute a prompt template.

### Decorators and Attributes

- `@kernel_function` (Python) -- Decorate a method to expose it as a kernel function.
- `[KernelFunction]` (C#) -- Attribute equivalent for C# methods.
- `@kernel_function(description="...", name="...")` -- Provide metadata for AI function discovery.

### Chat Completion

- `ChatCompletionService.get_chat_message_content(chat_history, settings, kernel)` -- Generate a completion from conversation history.
- `AzureChatCompletion(deployment_name, endpoint, api_key)` -- Azure OpenAI chat connector.
- `OpenAIChatCompletion(model_id, api_key)` -- OpenAI direct chat connector.

### Settings

- `PromptExecutionSettings(function_choice_behavior=FunctionChoiceBehavior.Auto())` -- Enable automatic function calling.
- `FunctionChoiceBehavior.Auto()` -- Let the model decide when to call functions.
- `FunctionChoiceBehavior.Required()` -- Force the model to call a function.
- `FunctionChoiceBehavior.None()` -- Disable function calling.

## Configuration and Customization

### AI Service Registration

Multiple AI services can be registered with different service identifiers. The kernel selects the appropriate service based on explicit request or configured selectors:

```python
kernel.add_service(
    AzureChatCompletion(
        deployment_name="gpt-4o",
        endpoint="https://my-resource.openai.azure.com/",
        api_key="...",
    ),
    service_id="gpt4o",
)

kernel.add_service(
    AzureChatCompletion(
        deployment_name="gpt-4o-mini",
        endpoint="https://my-resource.openai.azure.com/",
        api_key="...",
    ),
    service_id="gpt4o-mini",
)
```

### Function Choice Behavior

Control how the AI model interacts with registered plugins:

```python
from semantic_kernel.connectors.ai.open_ai import OpenAIChatPromptExecutionSettings
from semantic_kernel.connectors.ai.function_choice_behavior import FunctionChoiceBehavior

settings = OpenAIChatPromptExecutionSettings(
    function_choice_behavior=FunctionChoiceBehavior.Auto()
)
```

### Filters

Register pre- and post-invocation filters for prompt rendering and function execution:

```python
@kernel.filter("function_invocation")
async def my_filter(context, next):
    # Pre-invocation logic
    await next(context)
    # Post-invocation logic
```

## Integration Patterns

### Adding Existing Code as a Plugin

```python
from semantic_kernel.functions import kernel_function

class WeatherPlugin:
    @kernel_function(description="Get current weather for a city")
    def get_weather(self, city: str) -> str:
        # Call weather API
        return f"Weather in {city}: 72F, sunny"

kernel.add_plugin(WeatherPlugin(), plugin_name="Weather")
```

### Importing an OpenAPI Plugin

```python
await kernel.add_plugin_from_openapi(
    plugin_name="PetStore",
    openapi_document_path="https://petstore.swagger.io/v2/swagger.json",
)
```

### Creating an MCP Server from Kernel Functions

Kernel functions can be exposed as MCP tools through the built-in MCP server support, allowing external MCP clients to discover and invoke them over SSE or stdio transports.

### Chaining with Chat History

```python
from semantic_kernel.contents import ChatHistory

history = ChatHistory()
history.add_user_message("What is the weather in Seattle?")

result = await chat_service.get_chat_message_content(
    chat_history=history,
    settings=settings,
    kernel=kernel,
)
history.add_assistant_message(str(result))
```

## Examples

### Basic Chat Completion with Function Calling (Python)

```python
from semantic_kernel import Kernel
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
from semantic_kernel.connectors.ai.open_ai import OpenAIChatPromptExecutionSettings
from semantic_kernel.connectors.ai.function_choice_behavior import FunctionChoiceBehavior
from semantic_kernel.contents import ChatHistory
from semantic_kernel.functions import kernel_function


class TimePlugin:
    @kernel_function(description="Get the current time")
    def get_time(self) -> str:
        from datetime import datetime
        return datetime.now().strftime("%H:%M:%S")


kernel = Kernel()
kernel.add_service(
    AzureChatCompletion(
        deployment_name="gpt-4o",
        endpoint="https://my-resource.openai.azure.com/",
        api_key="my-api-key",
    )
)
kernel.add_plugin(TimePlugin(), plugin_name="Time")

settings = OpenAIChatPromptExecutionSettings(
    function_choice_behavior=FunctionChoiceBehavior.Auto()
)

history = ChatHistory()
history.add_user_message("What time is it right now?")

chat_service = kernel.get_service(type=AzureChatCompletion)
result = await chat_service.get_chat_message_content(
    chat_history=history,
    settings=settings,
    kernel=kernel,
)
print(result)
```

### Building and Invoking a Kernel (C#)

```csharp
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;

var builder = Kernel.CreateBuilder();
builder.AddAzureOpenAIChatCompletion("gpt-4o", endpoint, apiKey);
builder.Plugins.AddFromType<TimePlugin>();
Kernel kernel = builder.Build();

var chatService = kernel.GetRequiredService<IChatCompletionService>();
var history = new ChatHistory();
history.AddUserMessage("What time is it?");

var settings = new OpenAIPromptExecutionSettings
{
    FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
};

var result = await chatService.GetChatMessageContentAsync(
    history, settings, kernel
);
Console.WriteLine(result);
```

## Limitations and Considerations

- **C# as the primary language.** The C# SDK receives features first, with Python and Java implementations following. Feature parity across languages is not guaranteed at any given point.
- **Azure-centric defaults.** While the SDK supports multiple providers, the documentation, examples, and integration patterns are heavily oriented toward Azure OpenAI, requiring additional effort for non-Azure deployments.
- **Plugin discovery complexity.** Large numbers of registered plugins can overwhelm the AI model's function-calling context window, requiring manual curation of which plugins are available per request.
- **Learning curve for non-.NET developers.** The DI patterns and builder APIs are idiomatic to .NET, which may feel unfamiliar to Python or Java developers.
- **MCP support is recent.** MCP server creation from kernel functions is a new feature and may have limitations in transport support, error handling, or documentation maturity compared to established features.

## Changelog Highlights

- **1.0 GA.** Stable release with commitment to non-breaking changes. Kernel, plugins, and AI connectors finalized.
- **Process Framework.** Added support for defining multi-step AI workflows as state machines.
- **MCP Server Support.** Kernel functions can be exposed as MCP tools over SSE and stdio transports.
- **Anthropic Connector.** Added AI connector for Anthropic Claude models.
- **Google Connector.** Added AI connector for Google AI models.
- **Filters and Hooks.** Pre- and post-invocation filters for both prompt rendering and function execution.

## Citations

- [1] [Semantic Kernel Documentation](https://learn.microsoft.com/en-us/semantic-kernel/)
- [2] [Semantic Kernel Kernel Concept](https://learn.microsoft.com/en-us/semantic-kernel/concepts/kernel)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

semantic kernel, microsoft semantic kernel, c# llm sdk, dotnet ai, java llm sdk, kernel, kernel function, plugin, ai connector, azure openai, function choice behavior, openapi plugin, m365 copilot plugin, hipaa soc2 gdpr, opentelemetry, dependency injection, service provider, filters and hooks, prompt templates, process framework, mcp server from kernel functions, multi-language sdk, enterprise middleware, non-breaking changes

### Verb-Noun Tasks

- Build a `Kernel` and register an Azure OpenAI chat completion service
- Expose existing C# or Python code as a kernel function with `@kernel_function`
- Import a REST API as a plugin from its OpenAPI specification
- Register multiple AI services and select between them per request
- Configure `FunctionChoiceBehavior.Auto()` for automatic function calling
- Add pre/post invocation filters for content moderation and telemetry
- Emit OpenTelemetry traces for every prompt and function call
- Expose kernel functions as an MCP server over SSE or stdio
- Use the same SDK across C#, Python, and Java teams
- Compose a Process Framework state machine across plugins

### User Intent Phrases

- "How do I add AI to my existing C# or .NET application?"
- "How do I integrate Azure OpenAI with dependency injection?"
- "How do I expose existing business code as AI-callable functions?"
- "How do I make my REST APIs callable by an LLM?"
- "How do I build an enterprise AI assistant with HIPAA/SOC 2/GDPR compliance?"
- "How do I swap between OpenAI, Azure OpenAI, Anthropic, and Google in .NET?"
- "How do I add content moderation filters to every LLM call?"
- "How do I trace LLM calls with OpenTelemetry?"
- "How do I turn my kernel functions into an MCP server?"
- "How do I build a Microsoft 365 Copilot plugin?"
- "How do I write the same agent code in C# and Python?"

### Problem Statements

- Need to add AI to a large existing .NET / Java / Python codebase without rewriting business logic
- Enterprise compliance (HIPAA, SOC 2, GDPR) is required and rapid breaking framework changes are unacceptable
- Multiple teams use different languages and need a consistent SDK across them
- Existing REST APIs (and Microsoft 365 Copilot plugins) must be reusable by AI agents
- Need fine-grained content moderation and observability hooks at every step of the pipeline
- C# feature parity vs Python and Java lags and creates surprises
- Plugin discovery overflows the model's function-calling context window when many plugins are registered

### When to Pick This

- Pick this when you are in a .NET / Java / multi-language enterprise environment and need first-class C#, Python, and Java SDKs with the same abstractions
- Pick this over LangChain when .NET / Java parity, DI patterns, and enterprise compliance matter more than the broadest Python ecosystem
- Pick this over LangGraph when plugin-based function orchestration fits better than graph state machines
- Pick this over CrewAI when you want a kernel-and-plugins architecture instead of role-based crews
- Pick this over AutoGen when you want single-kernel plugin orchestration rather than multi-agent conversations (both are Microsoft, but this is the enterprise app-integration SDK)
- Pick this over Pydantic AI when multi-language SDKs, OpenTelemetry, and Azure-centric deployment matter more than Pydantic-typed Python agents
- Pick this over smolagents when enterprise middleware with filters, hooks, and compliance is required, not minimal code-first agents
- Pick this over ADK when your stack is anchored to Microsoft/Azure rather than Google/Vertex AI
- Pick this when OpenAPI-as-plugin and Microsoft 365 Copilot plugin compatibility are deliverables

### Related Terms and Aliases

- Kernel (DI container + pipeline)
- Kernel function (`@kernel_function`, `[KernelFunction]`)
- Plugin (group of functions)
- AI connector (provider abstraction)
- Function Choice Behavior (Auto / Required / None)
- Pre/post invocation filters
- Process Framework (state-machine workflows)
- OpenAPI plugin
- Microsoft 365 Copilot plugin format
- MCP server creation from kernel functions
- Enterprise AI middleware
- Multi-language SDK (C#, Python, Java)
