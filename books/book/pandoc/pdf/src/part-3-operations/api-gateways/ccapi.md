[Header 1 ("ccapi", [], []) [Str "ccapi"], BlockQuote [Para [Str "Unified AI API gateway for 100+ models with OpenAI-compatible endpoint"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API Gateways & Model Routing"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "API"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "No"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "N/A"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://ccapi.ai/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "CCAPI is a multimodal AI API gateway that aggregates multiple providers under a single OpenAI-compatible endpoint. The platform routes requests to over 100 models across seven or more providers spanning four modalities: text, image, audio, and video. CCAPI's core value proposition is migration simplicity -- existing code targeting the OpenAI API can be redirected to CCAPI by changing only the base URL to ", Code ("", [], []) "https://api.ccapi.ai/v1", Str " and supplying a CCAPI API key. Smart routing automatically switches between providers on failure with approximately 120 milliseconds of failover latency, maintaining a reported 99.9% success rate. The service operates on a pay-per-use billing model denominated in United States Dollars (USD) with no subscriptions or credit conversion. ", Str "[", Str "1", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], BulletList [[Plain [Strong [Str "Unified Endpoint"], Str ": A single OpenAI-compatible REST API base URL (", Code ("", [], []) "https://api.ccapi.ai/v1", Str ") that fronts all supported providers and modalities. Developers interact with one API surface regardless of whether the underlying model is served by OpenAI, Anthropic, Google, DeepSeek, or another provider."]], [Plain [Strong [Str "Smart Routing"], Str ": An automatic failover mechanism that detects provider downtime or errors and reroutes requests to alternative providers. The failover occurs in approximately 120 milliseconds, which is transparent to the caller. This operates across multiple retry layers to sustain the 99.9% success rate target."]], [Plain [Strong [Str "Multimodal Support"], Str ": CCAPI supports four modalities through dedicated endpoints -- text (chat completions), image generation, audio (text-to-speech), and video generation -- all accessible under the same base URL and authentication scheme."]], [Plain [Strong [Str "Provider-Prefixed Model Identifiers"], Str ": Models are referenced using a ", Code ("", [], []) "provider/model", Str " format (for example, ", Code ("", [], []) "anthropic/claude-4.6", Str " or ", Code ("", [], []) "bytedance/seedance-2", Str "), which disambiguates models across providers and allows explicit routing to a specific backend."]], [Plain [Strong [Str "Custom Providers"], Str ": Users can configure additional OpenAI-compatible providers with their own API keys and endpoints, extending CCAPI beyond its built-in provider catalog."]], [Plain [Strong [Str "Pay-Per-Use Billing"], Str ": No subscriptions, no credit conversion gimmicks. A $100 deposit equals $100 of usable balance, with real-time cost tracking via the usage dashboard."]]], Header 2 ("installation-and-setup", ["unnumbered", "unlisted"], []) [Str "Installation and Setup"], Para [Str "CCAPI is a hosted API service with no local installation required. Integration uses existing OpenAI-compatible SDKs."], Header 3 ("authentication", ["unnumbered", "unlisted"], []) [Str "Authentication"], Para [Str "CCAPI uses bearer token authentication. API keys follow the ", Code ("", [], []) "sk-ccapi-...", Str " prefix convention and are obtained from the CCAPI dashboard."], CodeBlock ("", ["bash"], []) "export CCAPI_API_KEY=\"sk-ccapi-your-key-here\"
", Para [Str "All requests must include the ", Code ("", [], []) "Authorization: Bearer <key>", Str " header."], Header 3 ("python-openai-sdk", ["unnumbered", "unlisted"], []) [Str "Python (OpenAI SDK)"], CodeBlock ("", ["bash"], []) "pip install openai
", CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    base_url=\"https://api.ccapi.ai/v1\",
    api_key=\"sk-ccapi-...\"
)
", Header 3 ("nodejs-openai-sdk", ["unnumbered", "unlisted"], []) [Str "Node.js (OpenAI SDK)"], CodeBlock ("", ["bash"], []) "npm install openai
", CodeBlock ("", ["javascript"], []) "import OpenAI from \"openai\";

const client = new OpenAI({
    baseURL: \"https://api.ccapi.ai/v1\",
    apiKey: \"sk-ccapi-...\",
});
", Header 3 ("curl", ["unnumbered", "unlisted"], []) [Str "cURL"], CodeBlock ("", ["bash"], []) "curl -X POST \"https://api.ccapi.ai/v1/chat/completions\" \\
  -H \"Authorization: Bearer $CCAPI_API_KEY\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"model\": \"anthropic/claude-4.6\",
    \"messages\": [{\"role\": \"user\", \"content\": \"Hello\"}]
  }'
", Para [Str "Any HTTP client or SDK that speaks REST and supports the OpenAI chat completions format works with CCAPI by pointing to the ", Code ("", [], []) "https://api.ccapi.ai/v1", Str " base URL. ", Str "[", Str "1", Str "]"], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "CCAPI's architecture consists of three logical layers:"], OrderedList (1, DefaultStyle, DefaultDelim) [[Plain [Strong [Str "API Gateway Layer"], Str ": An OpenAI-compatible REST API that accepts requests at ", Code ("", [], []) "https://api.ccapi.ai/v1", Str ". The gateway handles authentication (bearer tokens), request validation, rate management, and response formatting. All four modality endpoints (chat completions, image generation, audio, video generation) share the same gateway infrastructure and authentication scheme."]], [Plain [Strong [Str "Smart Routing Layer"], Str ": A routing and failover engine that sits between the gateway and upstream providers. When a request targets a specific provider-model pair, the routing layer forwards it to that provider. If the provider returns an error or is unreachable, the routing layer automatically retries with an alternative provider capable of serving an equivalent model, with failover latency of approximately 120 milliseconds. Multiple retry layers ensure high availability."]], [Plain [Strong [Str "Provider Integration Layer"], Str ": Connections to upstream AI providers (OpenAI, Anthropic, Google, DeepSeek, ByteDance, Kuaishou, Zhipu AI, MiniMax, Moonshot) plus user-configured custom providers. Each integration translates between the unified CCAPI request format and the provider's native API, handling authentication, request mapping, and response normalization."]]], Para [Str "The system is monitored 24/7 with real-time tracking of latency, success rates, and provider health across all four modalities."], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], BulletList [[Plain [Strong [Str "Chat Completions"], Str ": Standard text generation via ", Code ("", [], []) "/v1/chat/completions", Str " supporting streaming (Server-Sent Events (SSE)), function/tool calling, JSON response mode, temperature and top-p sampling, stop sequences, and maximum token limits. ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Extended Thinking"], Str ": A ", Code ("", [], []) "thinking", Str " parameter (", Code ("", [], []) "{\"type\": \"enabled\"}", Str ") activates step-by-step reasoning mode on supported models, returning intermediate reasoning content alongside the final response. ", Str "[", Str "2", Str "]"]], [Plain [Strong [Str "Vision and Image Input"], Str ": Multimodal models accept image inputs within the messages array for Optical Character Recognition (OCR), image analysis, and visual question answering."]], [Plain [Strong [Str "Image Generation"], Str ": Endpoint at ", Code ("", [], []) "/v1/images/generations", Str " for generating images from text prompts through supported providers."]], [Plain [Strong [Str "Audio (Text-to-Speech)"], Str ": Endpoint at ", Code ("", [], []) "/v1/audio/speech", Str " for converting text to audio using available speech synthesis models."]], [Plain [Strong [Str "Video Generation"], Str ": Endpoint at ", Code ("", [], []) "/v1/video/generations", Str " for generating video content. Supported models include Seedance 2.0 (ByteDance) and Kling 3.0 (Kuaishou)."]], [Plain [Strong [Str "Prompt Caching"], Str ": Repeated prompt prefixes (such as system prompts reused across conversations) receive reduced pricing, lowering cost for high-volume applications with shared context."]], [Plain [Strong [Str "Function/Tool Calling"], Str ": Tool definitions can be passed via the ", Code ("", [], []) "tools", Str " parameter with ", Code ("", [], []) "tool_choice", Str " controlling invocation behavior (", Code ("", [], []) "none", Str ", ", Code ("", [], []) "auto", Str ", ", Code ("", [], []) "required", Str ")."]], [Plain [Strong [Str "Usage Dashboard"], Str ": Real-time monitoring of costs, latency metrics, success rates, and per-request breakdowns. Tracks spending per model and per provider."]], [Plain [Strong [Str "Custom Provider Configuration"], Str ": Users can add any OpenAI-compatible provider with their own API keys, extending the gateway beyond the built-in provider catalog."]]], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], BulletList [[Plain [Strong [Str "Provider Migration"], Str ": Teams switching from one LLM provider to another can reroute by changing only the model identifier, with no SDK or integration code changes required. The OpenAI-compatible interface means the calling code remains identical."]], [Plain [Strong [Str "High-Availability AI Applications"], Str ": Production systems that cannot tolerate provider outages benefit from smart routing, which automatically fails over to alternative providers within 120 milliseconds."]], [Plain [Strong [Str "Multimodal Pipelines"], Str ": Applications that need text, image, audio, and video generation from a single integration point rather than maintaining separate SDKs and authentication for each provider."]], [Plain [Strong [Str "Cost Optimization"], Str ": Pay-per-use pricing with transparent USD billing and the ability to route to cost-effective providers (for example, DeepSeek V4 at $0.27 per million tokens) for workloads where the lowest-cost model is sufficient."]], [Plain [Strong [Str "Video Generation Access"], Str ": Teams needing access to Chinese AI video models (Seedance 2.0, Kling 3.0) through a familiar OpenAI-compatible interface without managing direct integrations with ByteDance or Kuaishou APIs."]], [Plain [Strong [Str "Prototyping and Evaluation"], Str ": Rapidly testing different models from different providers (GPT-5.2, Claude 4.6, DeepSeek V4, GLM-5) against the same prompts by changing only the model parameter, enabling quick comparison without provider-specific setup."]]], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("chat-completions", ["unnumbered", "unlisted"], []) [Str "Chat Completions"], Para [Strong [Str "Endpoint"], Str ": ", Code ("", [], []) "POST /v1/chat/completions"], Para [Strong [Str "Parameters"], Str ":"], BulletList [[Plain [Code ("", [], []) "model", Str " (required): Model identifier in ", Code ("", [], []) "provider/model", Str " format (e.g., ", Code ("", [], []) "anthropic/claude-4.6", Str ")."]], [Plain [Code ("", [], []) "messages", Str " (required): Array of message objects with ", Code ("", [], []) "role", Str " (", Code ("", [], []) "system", Str ", ", Code ("", [], []) "user", Str ", ", Code ("", [], []) "assistant", Str ", ", Code ("", [], []) "tool", Str ") and ", Code ("", [], []) "content", Str " fields."]], [Plain [Code ("", [], []) "stream", Str ": Boolean to enable SSE streaming of partial responses."]], [Plain [Code ("", [], []) "temperature", Str ": Sampling temperature, range ", Code ("", [], []) "0.0", Str " to ", Code ("", [], []) "2.0", Str ", default ", Code ("", [], []) "1.0", Str "."]], [Plain [Code ("", [], []) "top_p", Str ": Nucleus sampling threshold, range ", Code ("", [], []) "0.0", Str " to ", Code ("", [], []) "1.0", Str ", default ", Code ("", [], []) "1.0", Str "."]], [Plain [Code ("", [], []) "max_tokens", Str ": Maximum tokens in the generated response."]], [Plain [Code ("", [], []) "stop", Str ": String or array of stop sequences."]], [Plain [Code ("", [], []) "tools", Str ": Array of function/tool definitions for tool calling."]], [Plain [Code ("", [], []) "tool_choice", Str ": Control tool invocation (", Code ("", [], []) "none", Str ", ", Code ("", [], []) "auto", Str ", ", Code ("", [], []) "required", Str ")."]], [Plain [Code ("", [], []) "response_format", Str ": ", Code ("", [], []) "{\"type\": \"json_object\"}", Str " to enforce JSON output."]], [Plain [Code ("", [], []) "thinking", Str ": ", Code ("", [], []) "{\"type\": \"enabled\"}", Str " for extended reasoning mode."]]], Para [Strong [Str "Response"], Str ": Chat completion object containing generated content, token usage statistics, and optional tool calls or reasoning content."], Para [Strong [Str "Error Codes"], Str ":"], BulletList [[Plain [Code ("", [], []) "400", Str ": Bad request (malformed parameters)."]], [Plain [Code ("", [], []) "402", Str ": Insufficient account balance."]], [Plain [Code ("", [], []) "500", Str ": Server error."]]], Header 3 ("image-generation", ["unnumbered", "unlisted"], []) [Str "Image Generation"], Para [Strong [Str "Endpoint"], Str ": ", Code ("", [], []) "POST /v1/images/generations"], Header 3 ("audio-text-to-speech", ["unnumbered", "unlisted"], []) [Str "Audio (Text-to-Speech)"], Para [Strong [Str "Endpoint"], Str ": ", Code ("", [], []) "POST /v1/audio/speech"], Header 3 ("video-generation", ["unnumbered", "unlisted"], []) [Str "Video Generation"], Para [Strong [Str "Endpoint"], Str ": ", Code ("", [], []) "POST /v1/video/generations"], Header 3 ("available-models-selected", ["unnumbered", "unlisted"], []) [Str "Available Models (Selected)"], Para [Strong [Str "Text Generation"], Str ":"], BulletList [[Plain [Code ("", [], []) "openai/gpt-5.2", Str " -- OpenAI GPT-5.2 ($2.50/$10.00 per 1M input/output tokens)"]], [Plain [Code ("", [], []) "anthropic/claude-opus-4-6", Str " -- Anthropic Claude Opus 4.6, 200K context ($2.50/$12.50 per 1M tokens)"]], [Plain [Code ("", [], []) "anthropic/claude-sonnet-4-6", Str " -- Anthropic Claude Sonnet 4.6, 200K context ($1.50/$7.50 per 1M tokens)"]], [Plain [Code ("", [], []) "anthropic/claude-haiku-4-5", Str " -- Anthropic Claude Haiku 4.5, 200K context ($0.50/$2.50 per 1M tokens)"]], [Plain [Code ("", [], []) "deepseek/deepseek-v4", Str " -- DeepSeek V4 ($0.27/1M tokens)"]], [Plain [Code ("", [], []) "zhipu/glm-5", Str " -- Zhipu AI GLM-5 ($0.40/1M tokens)"]]], Para [Strong [Str "Video Generation"], Str ":"], BulletList [[Plain [Code ("", [], []) "bytedance/seedance-2", Str " -- ByteDance Seedance 2.0 ($0.34/second)"]], [Plain [Code ("", [], []) "kuaishou/kling-3.0", Str " -- Kuaishou Kling 3.0 ($0.39/video)"]]], Para [Str "CCAPI advertises up to 50% savings compared to direct provider pricing for certain models (notably Anthropic Claude models). ", Str "[", Str "1", Str "]", Str " ", Str "[", Str "2", Str "]"], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("base-url-configuration", ["unnumbered", "unlisted"], []) [Str "Base URL Configuration"], Para [Str "The single required configuration change for any OpenAI SDK-based application:"], CodeBlock ("", ["python"], []) "# Python
client = OpenAI(
    base_url=\"https://api.ccapi.ai/v1\",
    api_key=\"sk-ccapi-...\"
)
", CodeBlock ("", ["javascript"], []) "// Node.js
const client = new OpenAI({
    baseURL: \"https://api.ccapi.ai/v1\",
    apiKey: \"sk-ccapi-...\",
});
", Header 3 ("model-selection", ["unnumbered", "unlisted"], []) [Str "Model Selection"], Para [Str "Models are selected via the ", Code ("", [], []) "model", Str " parameter using provider-prefixed identifiers:"], CodeBlock ("", ["python"], []) "# Route to Anthropic Claude
response = client.chat.completions.create(
    model=\"anthropic/claude-4.6\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}]
)

# Route to DeepSeek
response = client.chat.completions.create(
    model=\"deepseek/deepseek-v4\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}]
)
", Header 3 ("custom-providers", ["unnumbered", "unlisted"], []) [Str "Custom Providers"], Para [Str "Users can configure additional OpenAI-compatible providers through the CCAPI dashboard, supplying their own API keys and endpoint URLs. This allows routing through CCAPI's unified interface to providers not in the built-in catalog."], Header 3 ("usage-dashboard", ["unnumbered", "unlisted"], []) [Str "Usage Dashboard"], Para [Str "The dashboard provides real-time visibility into:"], BulletList [[Plain [Str "Per-request cost breakdown"]], [Plain [Str "Latency metrics per model and provider"]], [Plain [Str "Success rate monitoring"]], [Plain [Str "API key management"]], [Plain [Str "Account balance tracking"]]], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("drop-in-openai-replacement", ["unnumbered", "unlisted"], []) [Str "Drop-In OpenAI Replacement"], Para [Str "The most common integration pattern requires changing only two values in existing OpenAI SDK code:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

# Before (direct OpenAI)
# client = OpenAI(api_key=\"sk-openai-...\")

# After (via CCAPI)
client = OpenAI(
    base_url=\"https://api.ccapi.ai/v1\",
    api_key=\"sk-ccapi-...\"
)

# All existing code works unchanged
response = client.chat.completions.create(
    model=\"openai/gpt-5.2\",
    messages=[{\"role\": \"user\", \"content\": \"Explain quantum computing.\"}]
)
", Header 3 ("multi-provider-switching", ["unnumbered", "unlisted"], []) [Str "Multi-Provider Switching"], Para [Str "A single client instance can target different providers by changing only the model parameter:"], CodeBlock ("", ["python"], []) "models = [
    \"openai/gpt-5.2\",
    \"anthropic/claude-4.6\",
    \"deepseek/deepseek-v4\",
]
for model in models:
    response = client.chat.completions.create(
        model=model,
        messages=[{\"role\": \"user\", \"content\": \"Summarize this document.\"}]
    )
", Header 3 ("framework-integration", ["unnumbered", "unlisted"], []) [Str "Framework Integration"], Para [Str "Any framework built on the OpenAI SDK (LangChain, LlamaIndex, CrewAI, and others) can route through CCAPI by configuring the base URL at the client level. No framework-specific adapters are needed."], Header 3 ("resthttp-client-integration", ["unnumbered", "unlisted"], []) [Str "REST/HTTP Client Integration"], Para [Str "Any language or framework with HTTP client capabilities can call CCAPI directly:"], CodeBlock ("", ["bash"], []) "curl -X POST \"https://api.ccapi.ai/v1/chat/completions\" \\
  -H \"Authorization: Bearer sk-ccapi-...\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"model\": \"anthropic/claude-sonnet-4-6\",
    \"messages\": [{\"role\": \"user\", \"content\": \"Hello\"}],
    \"stream\": false
  }'
", Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("basic-chat-completion-python", ["unnumbered", "unlisted"], []) [Str "Basic Chat Completion (Python)"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    base_url=\"https://api.ccapi.ai/v1\",
    api_key=\"sk-ccapi-...\"
)

response = client.chat.completions.create(
    model=\"anthropic/claude-4.6\",
    messages=[{\"role\": \"user\", \"content\": \"What is machine learning?\"}]
)
print(response.choices[0].message.content)
", Header 3 ("streaming-response-nodejs", ["unnumbered", "unlisted"], []) [Str "Streaming Response (Node.js)"], CodeBlock ("", ["javascript"], []) "import OpenAI from \"openai\";

const client = new OpenAI({
    baseURL: \"https://api.ccapi.ai/v1\",
    apiKey: \"sk-ccapi-...\",
});

const stream = await client.chat.completions.create({
    model: \"anthropic/claude-4.6\",
    messages: [
        { role: \"system\", content: \"You are a helpful assistant.\" },
        { role: \"user\", content: \"Explain recursion step by step.\" },
    ],
    stream: true,
});

for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content;
    if (content) process.stdout.write(content);
}
", Header 3 ("tool-calling-python", ["unnumbered", "unlisted"], []) [Str "Tool Calling (Python)"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    base_url=\"https://api.ccapi.ai/v1\",
    api_key=\"sk-ccapi-...\"
)

tools = [
    {
        \"type\": \"function\",
        \"function\": {
            \"name\": \"get_weather\",
            \"description\": \"Get current weather for a location\",
            \"parameters\": {
                \"type\": \"object\",
                \"properties\": {
                    \"location\": {\"type\": \"string\", \"description\": \"City name\"}
                },
                \"required\": [\"location\"]
            }
        }
    }
]

response = client.chat.completions.create(
    model=\"openai/gpt-5.2\",
    messages=[{\"role\": \"user\", \"content\": \"What is the weather in Tokyo?\"}],
    tools=tools,
    tool_choice=\"auto\"
)
", Header 3 ("extended-thinking-curl", ["unnumbered", "unlisted"], []) [Str "Extended Thinking (cURL)"], CodeBlock ("", ["bash"], []) "curl -X POST \"https://api.ccapi.ai/v1/chat/completions\" \\
  -H \"Authorization: Bearer sk-ccapi-...\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"model\": \"anthropic/claude-4.6\",
    \"messages\": [{\"role\": \"user\", \"content\": \"Solve this step by step: 23 * 47 + 19\"}],
    \"thinking\": {\"type\": \"enabled\"}
  }'
", Header 3 ("video-generation-curl", ["unnumbered", "unlisted"], []) [Str "Video Generation (cURL)"], CodeBlock ("", ["bash"], []) "curl -X POST \"https://api.ccapi.ai/v1/video/generations\" \\
  -H \"Authorization: Bearer sk-ccapi-...\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"model\": \"bytedance/seedance-2\",
    \"prompt\": \"A serene mountain lake at sunrise with mist rolling over the water\"
  }'
", Header 3 ("json-mode-response-python", ["unnumbered", "unlisted"], []) [Str "JSON Mode Response (Python)"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(
    base_url=\"https://api.ccapi.ai/v1\",
    api_key=\"sk-ccapi-...\"
)

response = client.chat.completions.create(
    model=\"deepseek/deepseek-v4\",
    messages=[{\"role\": \"user\", \"content\": \"List three programming languages with their use cases.\"}],
    response_format={\"type\": \"json_object\"}
)
", Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Closed-Source Service"], Str ": CCAPI is a proprietary hosted service with no self-hosted or on-premises deployment option. All requests transit through CCAPI's infrastructure, which introduces a dependency on their availability and data handling practices."]], [Plain [Strong [Str "Provider-Dependent Features"], Str ": Feature support (tool calling, vision, extended thinking, structured output) varies by upstream model and provider. Not all features are available across all models."]], [Plain [Strong [Str "No Model Fine-Tuning"], Str ": CCAPI is an inference routing layer, not a model training or fine-tuning platform. Fine-tuned models must be hosted by the upstream provider and accessed through CCAPI if that provider is supported."]], [Plain [Strong [Str "Latency Overhead"], Str ": Routing through an intermediary gateway adds network hops compared to calling a provider directly. While CCAPI reports sub-2-second latency, the additional hop may be noticeable for latency-sensitive applications where every millisecond matters."]], [Plain [Strong [Str "Rate Limits"], Str ": Rate limits and quotas are not publicly documented in detail. Throughput may be constrained by both CCAPI's gateway limits and the underlying provider's rate limits."]], [Plain [Strong [Str "Model Availability Lag"], Str ": New models released by providers may not be immediately available through CCAPI. There is an inherent delay between a provider launching a model and CCAPI integrating it."]], [Plain [Strong [Str "Limited Documentation"], Str ": As a newer service, CCAPI's public documentation is less extensive than established gateways. Detailed API reference for image, audio, and video endpoints is sparse compared to the chat completions documentation."]], [Plain [Strong [Str "Vendor Lock-In Risk"], Str ": While CCAPI uses an OpenAI-compatible interface (reducing switching cost), reliance on provider-prefixed model identifiers (", Code ("", [], []) "anthropic/claude-4.6", Str ") creates a CCAPI-specific naming convention that requires mapping if migrating to another gateway."]], [Plain [Strong [Str "Data Privacy"], Str ": All requests and responses pass through CCAPI's servers. Organizations with strict data residency or privacy requirements should evaluate CCAPI's data handling policies before routing sensitive workloads through the gateway."]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Str "Launch of unified multimodal API gateway supporting text, image, audio, and video through a single endpoint."]], [Plain [Str "Smart routing with automatic provider failover in approximately 120 milliseconds."]], [Plain [Str "Integration of video generation models Seedance 2.0 (ByteDance) and Kling 3.0 (Kuaishou)."]], [Plain [Str "Support for extended thinking mode on compatible models."]], [Plain [Str "Function/tool calling support across providers."]], [Plain [Str "Prompt caching with reduced pricing for repeated prefixes."]], [Plain [Str "Custom provider configuration allowing users to bring their own API keys."]], [Plain [Str "Real-time usage dashboard with cost, latency, and success rate monitoring."]], [Plain [Str "Claude model pricing at up to 50% below direct Anthropic pricing."]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " CCAPI Official Website - https://ccapi.ai/"]], [Plain [Str "[", Str "2", Str "]", Str " CCAPI API Documentation - https://docs.ccapi.ai/"]]]]