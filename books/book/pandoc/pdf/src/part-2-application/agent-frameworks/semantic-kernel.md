[Header 1 ("semantic-kernel", [], []) [Str "Semantic Kernel"], BlockQuote [Para [Str "Enterprise-grade, lightweight SDK for integrating Large Language Models (LLMs) into conventional applications across C#, Python, and Java, providing modular plugin architecture and AI service orchestration."]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, (ColWidth 0.1839080459770115)), (AlignDefault, (ColWidth 0.8160919540229885))] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Name"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Semantic Kernel"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Group"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Agent Frameworks"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Type"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Open Source"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "GitHub"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "microsoft/semantic-kernel"] ("https://github.com/microsoft/semantic-kernel", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Stars"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "27,283"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Strong [Str "Docs"]]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://learn.microsoft.com/en-us/semantic-kernel/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "Semantic Kernel is a lightweight, open-source SDK developed by Microsoft for integrating LLMs into C#, Python, and Java applications. It serves as enterprise-grade middleware for building AI agents, providing a structured approach to combining conventional code with AI model capabilities. The SDK reached version 1.0+ with a commitment to non-breaking changes, making it suitable for production deployments in regulated industries. Semantic Kernel is used internally by Microsoft and adopted by Fortune 500 companies, with compliance support for HIPAA, SOC 2, and GDPR."], Para [Str "The SDK follows a plugin-based architecture where developers expose existing code as functions that AI models can discover and invoke. This design allows teams to incrementally add AI capabilities to existing applications without rewriting business logic. Semantic Kernel handles prompt construction, AI service routing, response parsing, and function orchestration through a central kernel object that acts as both a Dependency Injection (DI) container and an execution pipeline."], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Kernel."], Str " The central orchestration object that manages all services and plugins. The kernel selects the appropriate AI service for a given request, builds prompts from templates and context, sends them to the AI model, and parses the responses. It supports events and middleware at each step of this pipeline, enabling cross-cutting concerns such as logging, telemetry, and content filtering. The kernel is modeled after the .NET Service Provider pattern, making it familiar to enterprise .NET developers."], Para [Strong [Str "Services."], Str " AI services (such as chat completion models) and infrastructure services (logging, HTTP clients, telemetry) registered with the kernel. Services are resolved through the kernel's DI container, allowing runtime selection of AI providers based on request characteristics. Multiple AI services can be registered simultaneously, with the kernel choosing the appropriate one per invocation."], Para [Strong [Str "Plugins."], Str " Components that AI services use to perform work in the real world. Plugins retrieve data, call APIs, manipulate files, and execute arbitrary business logic. Each plugin contains one or more kernel functions that are described with metadata so the AI model can understand when and how to invoke them. Plugins bridge the gap between the AI model's reasoning and the application's capabilities."], Para [Strong [Str "Functions."], Str " The individual callable units within plugins. Functions can be native code decorated with ", Code ("", [], []) "@kernel_function", Str " (Python) or equivalent attributes in C# and Java, or they can be prompt templates that the kernel renders and sends to an AI service. Native functions execute conventional code, while prompt functions generate AI completions. Both types are treated uniformly by the kernel's function-calling infrastructure."], Para [Strong [Str "AI Connectors."], Str " Integrations with AI model providers including OpenAI, Azure OpenAI, Google, and Anthropic. Connectors abstract the provider-specific API details behind a common interface, allowing developers to swap models without rewriting application code. Each connector handles authentication, request formatting, and response deserialization for its target provider."], Para [Strong [Str "Model Context Protocol (MCP) Server."], Str " A recent addition that allows developers to create MCP servers directly from kernel functions, exposing them over Server-Sent Events (SSE) and stdio transports. Prompt templates can also be exposed as MCP prompts, enabling interoperability with the broader MCP ecosystem."], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "Semantic Kernel follows a layered pipeline architecture centered on the kernel object:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "Request Layer."], Str " The application submits a prompt or function call request to the kernel."]], [Plain [Strong [Str "Service Selection."], Str " The kernel evaluates registered AI services and selects the appropriate one based on service selectors, model capabilities, or explicit configuration."]], [Plain [Strong [Str "Prompt Rendering."], Str " If the request involves a prompt template, the kernel renders it by injecting variables, function results, and context from the conversation history."]], [Plain [Strong [Str "Pre-Invocation Filters."], Str " Registered filters and hooks execute before the AI service call, enabling prompt modification, content safety checks, or request logging."]], [Plain [Strong [Str "AI Service Invocation."], Str " The selected AI connector sends the rendered prompt to the model provider and receives the response."]], [Plain [Strong [Str "Function Calling Loop."], Str " If the AI response includes function call requests, the kernel resolves the target plugin functions, executes them, appends the results to the conversation, and re-invokes the AI service. This loop continues until the model produces a final response without further function calls."]], [Plain [Strong [Str "Post-Invocation Filters."], Str " Registered filters execute after the AI response, enabling response modification, telemetry recording, or content filtering."]], [Plain [Strong [Str "Response Layer."], Str " The processed response is returned to the application."]]], Para [Str "The plugin system is flat by design. Plugins register with the kernel at startup, and their function metadata (descriptions, parameter types, return types) is automatically serialized into the AI model's function-calling schema. The kernel manages the serialization and deserialization of function arguments and return values transparently."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], BulletList [[Plain [Strong [Str "Multi-language support."], Str " First-class SDKs for C#, Python, and Java with consistent abstractions and API surface across all three languages."]], [Plain [Strong [Str "Plugin architecture."], Str " Expose existing code as AI-callable functions through simple decorators or attributes, without modifying the underlying business logic."]], [Plain [Strong [Str "OpenAPI specification support."], Str " Import OpenAPI specs as plugins, making any REST API available to the AI model. Compatible with Microsoft 365 Copilot plugin format."]], [Plain [Strong [Str "Model portability."], Str " Swap between OpenAI, Azure OpenAI, Google, and Anthropic models by changing the AI connector configuration without altering application code."]], [Plain [Strong [Str "Telemetry and observability."], Str " Built-in support for OpenTelemetry-compatible instrumentation, providing visibility into prompt rendering, AI service calls, and function execution."]], [Plain [Strong [Str "Hooks and filters."], Str " Pre- and post-invocation filters at both the prompt and function levels, enabling content moderation, logging, retry logic, and custom middleware."]], [Plain [Strong [Str "MCP server creation."], Str " Generate MCP-compliant servers from kernel functions, exposing them over SSE and stdio transports for interoperability with MCP clients."]], [Plain [Strong [Str "Prompt templates as MCP prompts."], Str " Semantic Kernel prompt templates can be published as MCP prompts, enabling external tools to discover and invoke them."]], [Plain [Strong [Str "Dependency injection integration."], Str " Native DI support in C# through the standard ", Code ("", [], []) "IServiceProvider", Str " pattern, and analogous service registration in Python and Java."]], [Plain [Strong [Str "Non-breaking changes commitment."], Str " Version 1.0+ maintains backward compatibility, providing stability for production deployments."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Enterprise AI assistants."], Str " Building internal copilots that combine LLM reasoning with access to corporate APIs, databases, and document repositories through plugins."]], [Plain [Strong [Str "Customer service automation."], Str " Orchestrating multi-step workflows where the AI agent queries knowledge bases, retrieves order information, and executes actions through function calling."]], [Plain [Strong [Str "Document processing pipelines."], Str " Combining AI-powered extraction and summarization with conventional data validation and storage logic."]], [Plain [Strong [Str "Multi-model orchestration."], Str " Routing different types of requests to specialized models (such as using a smaller model for classification and a larger model for generation) through the service selection mechanism."]], [Plain [Strong [Str "Legacy system AI integration."], Str " Wrapping existing REST APIs and code libraries as plugins to make them accessible to AI agents without rewriting the underlying systems."]], [Plain [Strong [Str "Compliance-sensitive deployments."], Str " Leveraging content filters and audit hooks for regulated industries that require logging and content moderation of all AI interactions."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("kernel", ["unnumbered", "unlisted"], []) [Str "Kernel"], BulletList [[Plain [Code ("", [], []) "Kernel()", Str " -- Create a new kernel instance (Python)."]], [Plain [Code ("", [], []) "Kernel.CreateBuilder()", Str " -- Create a kernel builder (C#)."]], [Plain [Code ("", [], []) "kernel.add_service(service)", Str " -- Register an AI or infrastructure service."]], [Plain [Code ("", [], []) "kernel.add_plugin(plugin, plugin_name)", Str " -- Register a plugin with the kernel."]], [Plain [Code ("", [], []) "kernel.invoke(function, arguments)", Str " -- Execute a kernel function with the given arguments."]], [Plain [Code ("", [], []) "kernel.invoke_prompt(prompt, arguments)", Str " -- Render and execute a prompt template."]]], Header 3 ("decorators-and-attributes", ["unnumbered", "unlisted"], []) [Str "Decorators and Attributes"], BulletList [[Plain [Code ("", [], []) "@kernel_function", Str " (Python) -- Decorate a method to expose it as a kernel function."]], [Plain [Code ("", [], []) "[KernelFunction]", Str " (C#) -- Attribute equivalent for C# methods."]], [Plain [Code ("", [], []) "@kernel_function(description=\"...\", name=\"...\")", Str " -- Provide metadata for AI function discovery."]]], Header 3 ("chat-completion", ["unnumbered", "unlisted"], []) [Str "Chat Completion"], BulletList [[Plain [Code ("", [], []) "ChatCompletionService.get_chat_message_content(chat_history, settings, kernel)", Str " -- Generate a completion from conversation history."]], [Plain [Code ("", [], []) "AzureChatCompletion(deployment_name, endpoint, api_key)", Str " -- Azure OpenAI chat connector."]], [Plain [Code ("", [], []) "OpenAIChatCompletion(model_id, api_key)", Str " -- OpenAI direct chat connector."]]], Header 3 ("settings", ["unnumbered", "unlisted"], []) [Str "Settings"], BulletList [[Plain [Code ("", [], []) "PromptExecutionSettings(function_choice_behavior=FunctionChoiceBehavior.Auto())", Str " -- Enable automatic function calling."]], [Plain [Code ("", [], []) "FunctionChoiceBehavior.Auto()", Str " -- Let the model decide when to call functions."]], [Plain [Code ("", [], []) "FunctionChoiceBehavior.Required()", Str " -- Force the model to call a function."]], [Plain [Code ("", [], []) "FunctionChoiceBehavior.None()", Str " -- Disable function calling."]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("ai-service-registration", ["unnumbered", "unlisted"], []) [Str "AI Service Registration"], Para [Str "Multiple AI services can be registered with different service identifiers. The kernel selects the appropriate service based on explicit request or configured selectors:"], CodeBlock ("", ["python"], []) "kernel.add_service(
    AzureChatCompletion(
        deployment_name=\"gpt-4o\",
        endpoint=\"https://my-resource.openai.azure.com/\",
        api_key=\"...\",
    ),
    service_id=\"gpt4o\",
)

kernel.add_service(
    AzureChatCompletion(
        deployment_name=\"gpt-4o-mini\",
        endpoint=\"https://my-resource.openai.azure.com/\",
        api_key=\"...\",
    ),
    service_id=\"gpt4o-mini\",
)
", Header 3 ("function-choice-behavior", ["unnumbered", "unlisted"], []) [Str "Function Choice Behavior"], Para [Str "Control how the AI model interacts with registered plugins:"], CodeBlock ("", ["python"], []) "from semantic_kernel.connectors.ai.open_ai import OpenAIChatPromptExecutionSettings
from semantic_kernel.connectors.ai.function_choice_behavior import FunctionChoiceBehavior

settings = OpenAIChatPromptExecutionSettings(
    function_choice_behavior=FunctionChoiceBehavior.Auto()
)
", Header 3 ("filters", ["unnumbered", "unlisted"], []) [Str "Filters"], Para [Str "Register pre- and post-invocation filters for prompt rendering and function execution:"], CodeBlock ("", ["python"], []) "@kernel.filter(\"function_invocation\")
async def my_filter(context, next):
    # Pre-invocation logic
    await next(context)
    # Post-invocation logic
", Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("adding-existing-code-as-a-plugin", ["unnumbered", "unlisted"], []) [Str "Adding Existing Code as a Plugin"], CodeBlock ("", ["python"], []) "from semantic_kernel.functions import kernel_function

class WeatherPlugin:
    @kernel_function(description=\"Get current weather for a city\")
    def get_weather(self, city: str) -> str:
        # Call weather API
        return f\"Weather in {city}: 72F, sunny\"

kernel.add_plugin(WeatherPlugin(), plugin_name=\"Weather\")
", Header 3 ("importing-an-openapi-plugin", ["unnumbered", "unlisted"], []) [Str "Importing an OpenAPI Plugin"], CodeBlock ("", ["python"], []) "await kernel.add_plugin_from_openapi(
    plugin_name=\"PetStore\",
    openapi_document_path=\"https://petstore.swagger.io/v2/swagger.json\",
)
", Header 3 ("creating-an-mcp-server-from-kernel-functions", ["unnumbered", "unlisted"], []) [Str "Creating an MCP Server from Kernel Functions"], Para [Str "Kernel functions can be exposed as MCP tools through the built-in MCP server support, allowing external MCP clients to discover and invoke them over SSE or stdio transports."], Header 3 ("chaining-with-chat-history", ["unnumbered", "unlisted"], []) [Str "Chaining with Chat History"], CodeBlock ("", ["python"], []) "from semantic_kernel.contents import ChatHistory

history = ChatHistory()
history.add_user_message(\"What is the weather in Seattle?\")

result = await chat_service.get_chat_message_content(
    chat_history=history,
    settings=settings,
    kernel=kernel,
)
history.add_assistant_message(str(result))
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-chat-completion-with-function-calling-python", ["unnumbered", "unlisted"], []) [Str "Basic Chat Completion with Function Calling (Python)"], CodeBlock ("", ["python"], []) "from semantic_kernel import Kernel
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
from semantic_kernel.connectors.ai.open_ai import OpenAIChatPromptExecutionSettings
from semantic_kernel.connectors.ai.function_choice_behavior import FunctionChoiceBehavior
from semantic_kernel.contents import ChatHistory
from semantic_kernel.functions import kernel_function


class TimePlugin:
    @kernel_function(description=\"Get the current time\")
    def get_time(self) -> str:
        from datetime import datetime
        return datetime.now().strftime(\"%H:%M:%S\")


kernel = Kernel()
kernel.add_service(
    AzureChatCompletion(
        deployment_name=\"gpt-4o\",
        endpoint=\"https://my-resource.openai.azure.com/\",
        api_key=\"my-api-key\",
    )
)
kernel.add_plugin(TimePlugin(), plugin_name=\"Time\")

settings = OpenAIChatPromptExecutionSettings(
    function_choice_behavior=FunctionChoiceBehavior.Auto()
)

history = ChatHistory()
history.add_user_message(\"What time is it right now?\")

chat_service = kernel.get_service(type=AzureChatCompletion)
result = await chat_service.get_chat_message_content(
    chat_history=history,
    settings=settings,
    kernel=kernel,
)
print(result)
", Header 3 ("building-and-invoking-a-kernel-c", ["unnumbered", "unlisted"], []) [Str "Building and Invoking a Kernel (C#)"], CodeBlock ("", ["csharp"], []) "using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;

var builder = Kernel.CreateBuilder();
builder.AddAzureOpenAIChatCompletion(\"gpt-4o\", endpoint, apiKey);
builder.Plugins.AddFromType<TimePlugin>();
Kernel kernel = builder.Build();

var chatService = kernel.GetRequiredService<IChatCompletionService>();
var history = new ChatHistory();
history.AddUserMessage(\"What time is it?\");

var settings = new OpenAIPromptExecutionSettings
{
    FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
};

var result = await chatService.GetChatMessageContentAsync(
    history, settings, kernel
);
Console.WriteLine(result);
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "C# as the primary language."], Str " The C# SDK receives features first, with Python and Java implementations following. Feature parity across languages is not guaranteed at any given point."]], [Plain [Strong [Str "Azure-centric defaults."], Str " While the SDK supports multiple providers, the documentation, examples, and integration patterns are heavily oriented toward Azure OpenAI, requiring additional effort for non-Azure deployments."]], [Plain [Strong [Str "Plugin discovery complexity."], Str " Large numbers of registered plugins can overwhelm the AI model's function-calling context window, requiring manual curation of which plugins are available per request."]], [Plain [Strong [Str "Learning curve for non-.NET developers."], Str " The DI patterns and builder APIs are idiomatic to .NET, which may feel unfamiliar to Python or Java developers."]], [Plain [Strong [Str "MCP support is recent."], Str " MCP server creation from kernel functions is a new feature and may have limitations in transport support, error handling, or documentation maturity compared to established features."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "1.0 GA."], Str " Stable release with commitment to non-breaking changes. Kernel, plugins, and AI connectors finalized."]], [Plain [Strong [Str "Process Framework."], Str " Added support for defining multi-step AI workflows as state machines."]], [Plain [Strong [Str "MCP Server Support."], Str " Kernel functions can be exposed as MCP tools over SSE and stdio transports."]], [Plain [Strong [Str "Anthropic Connector."], Str " Added AI connector for Anthropic Claude models."]], [Plain [Strong [Str "Google Connector."], Str " Added AI connector for Google AI models."]], [Plain [Strong [Str "Filters and Hooks."], Str " Pre- and post-invocation filters for both prompt rendering and function execution."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " ", Link ("", [], []) [Str "Semantic Kernel Documentation"] ("https://learn.microsoft.com/en-us/semantic-kernel/", "")]], [Plain [Str "[", Str "2", Str "]", Str " ", Link ("", [], []) [Str "Semantic Kernel Kernel Concept"] ("https://learn.microsoft.com/en-us/semantic-kernel/concepts/kernel", "")]]]]