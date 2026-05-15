[Header 1 ("litellm", [], []) [Str "LiteLLM"], BlockQuote [Para [Str "Python SDK and proxy for unified access to 100+ LLM APIs"]], Table ("", [], []) (Caption Nothing []) [(AlignDefault, ColWidthDefault), (AlignDefault, ColWidthDefault)] (TableHead ("", [], []) [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Field"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Value"]]]]) [(TableBody ("", [], []) (RowHeadColumns 0) [] [Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Group"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Model Gateways"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Type"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "SDK"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Open Source"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Yes"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "GitHub"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "BerriAI/litellm"] ("https://github.com/BerriAI/litellm", "")]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Stars"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "46980"]]], Row ("", [], []) [Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Str "Documentation"]], Cell ("", [], []) AlignDefault (RowSpan 0) (ColSpan 0) [Plain [Link ("", [], []) [Str "Official Docs"] ("https://docs.litellm.ai/docs/", "")]]]])] (TableFoot ("", [], []) []), Header 2 ("overview", ["unnumbered", "unlisted"], []) [Str "Overview"], Para [Str "LiteLLM is an open source Python SDK and proxy server that provides a unified interface for calling 100+ large language models using the OpenAI input/output format. It translates inputs to provider-specific endpoints and normalizes responses into a consistent format, allowing developers to switch between providers without rewriting application code. The project is maintained by BerriAI and supports providers including OpenAI, Anthropic, xAI, Google Vertex AI, Azure OpenAI, NVIDIA NIM, HuggingFace, Ollama, OpenRouter, Novita AI, and Vercel AI Gateway. ", Str "[", Str "1", Str "]"], Para [Str "LiteLLM ships as two components. The ", Strong [Str "Python SDK"], Str " embeds directly into applications and provides completion calls, retry/fallback logic, observability callbacks, and cost tracking. The ", Strong [Str "Proxy Server"], Str " (also called the LLM Gateway) runs as a standalone service that exposes an OpenAI-compatible API with authentication, multi-tenant cost tracking, virtual keys, rate limiting, load balancing, and an admin dashboard. Both components share the same underlying translation layer and provider support. ", Str "[", Str "1", Str "]"], Header 2 ("core-concepts", ["unnumbered", "unlisted"], []) [Str "Core Concepts"], Para [Strong [Str "Unified Completion Interface"], Str ": The ", Code ("", [], []) "completion()", Str " function accepts an OpenAI-style model identifier and messages array, translating the request to the target provider's native format. Responses are normalized to match OpenAI's chat completion structure regardless of the underlying provider. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Provider Prefixes"], Str ": Models are specified using a ", Code ("", [], []) "provider/model-name", Str " format (e.g., ", Code ("", [], []) "openai/gpt-4o", Str ", ", Code ("", [], []) "anthropic/claude-opus-4-6", Str ", ", Code ("", [], []) "azure/gpt-4o-eu", Str "). The prefix tells LiteLLM which translation layer to apply for the request and response. ", Str "[", Str "1", Str "]", Str "[", Str "4", Str "]"], Para [Strong [Str "Router"], Str ": The Router manages load balancing across multiple deployments of the same model. Deployments sharing the same ", Code ("", [], []) "model_name", Str " form a model group, and the Router selects among them using configurable strategies. The Router also handles cooldowns, retries, and fallbacks when deployments fail. ", Str "[", Str "3", Str "]"], Para [Strong [Str "Virtual Keys"], Str ": The Proxy Server issues virtual API keys that abstract over underlying provider credentials. Each virtual key can have its own spend budget, rate limits, and model access restrictions. Users and teams interact with virtual keys rather than raw provider API keys. ", Str "[", Str "5", Str "]"], Para [Strong [Str "Exception Mapping"], Str ": Provider-specific errors are mapped to OpenAI exception types (", Code ("", [], []) "AuthenticationError", Str ", ", Code ("", [], []) "RateLimitError", Str ", ", Code ("", [], []) "APIError", Str ", ", Code ("", [], []) "Timeout", Str ", ", Code ("", [], []) "NotFoundError", Str ", ", Code ("", [], []) "ServiceUnavailableError", Str ", ", Code ("", [], []) "ContentPolicyViolationError", Str "). All exceptions include ", Code ("", [], []) "status_code", Str ", ", Code ("", [], []) "message", Str ", and ", Code ("", [], []) "llm_provider", Str " attributes for debugging. ", Str "[", Str "6", Str "]"], Para [Strong [Str "Observability Callbacks"], Str ": Three callback types (input, success, failure) send telemetry to external platforms. Callbacks are configured declaratively by setting ", Code ("", [], []) "litellm.success_callback", Str " and ", Code ("", [], []) "litellm.failure_callback", Str " to lists of integration names. ", Str "[", Str "7", Str "]"], Header 2 ("architecture", ["unnumbered", "unlisted"], []) [Str "Architecture"], Para [Str "LiteLLM is organized into three layers:"], CodeBlock ("", [""], []) "SDK Layer              completion(), embedding(), image_generation()
    |                  Provider translation, response normalization
    |
Router Layer           Load balancing, retries, fallbacks, cooldowns
    |                  Model groups, deployment health tracking
    |
Proxy Layer            HTTP server, virtual keys, spend tracking
                       Admin dashboard, rate limiting, auth
", Para [Strong [Str "SDK Layer"], Str ": The core translation engine. Each provider has a handler that converts OpenAI-format requests into provider-native API calls and normalizes responses back to OpenAI format. The SDK is stateless and can be embedded directly into Python applications. ", Str "[", Str "1", Str "]"], Para [Strong [Str "Router Layer"], Str ": Sits on top of the SDK and manages multiple deployments. It tracks deployment health, enforces rate limits (Requests Per Minute (RPM) and Tokens Per Minute (TPM)), and applies routing strategies to distribute traffic. The Router operates in-process for SDK usage or as part of the Proxy Server. ", Str "[", Str "3", Str "]"], Para [Strong [Str "Proxy Layer"], Str ": A standalone HTTP server built on the Router. It adds authentication (master key and virtual keys), a PostgreSQL-backed spend tracking database, team and user management, and an admin dashboard. Clients interact with it using standard OpenAI SDKs pointed at the proxy's base URL. ", Str "[", Str "2", Str "]", Str "[", Str "5", Str "]"], Header 2 ("key-features-and-functionality", ["unnumbered", "unlisted"], []) [Str "Key Features and Functionality"], Para [Strong [Str "Unified Completion Calls"], Str ": Call any supported provider through a single function with consistent input/output format:"], CodeBlock ("", ["python"], []) "from litellm import completion

# OpenAI
response = completion(model=\"openai/gpt-4o\", messages=[{\"role\": \"user\", \"content\": \"Hi\"}])

# Anthropic
response = completion(model=\"anthropic/claude-opus-4-6\", messages=[{\"role\": \"user\", \"content\": \"Hi\"}])

# Azure OpenAI
response = completion(model=\"azure/gpt-4o-eu\", messages=[{\"role\": \"user\", \"content\": \"Hi\"}])
", Para [Str "[", Str "1", Str "]"], Para [Strong [Str "Streaming"], Str ": All providers support streaming via ", Code ("", [], []) "stream=True", Str ". Streamed responses return chunks in OpenAI's Server-Sent Events (SSE) format with token-level deltas and usage metadata:"], CodeBlock ("", ["python"], []) "response = completion(
    model=\"openai/gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Write a story\"}],
    stream=True,
)
for chunk in response:
    print(chunk.choices[0].delta.content or \"\", end=\"\")
", Para [Str "[", Str "1", Str "]"], Para [Strong [Str "Retry and Fallback Logic"], Str ": Configure automatic retries with exponential backoff and model fallbacks:"], CodeBlock ("", ["python"], []) "from litellm import completion

response = completion(
    model=\"openai/gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    num_retries=3,
    fallbacks=[\"anthropic/claude-sonnet-4-6\", \"azure/gpt-4o\"],
)
", Para [Str "[", Str "1", Str "]"], Para [Strong [Str "Cost Tracking"], Str ": LiteLLM calculates costs per request using built-in model pricing data. Custom per-token pricing can be specified per deployment:"], CodeBlock ("", ["python"], []) "response = completion(
    model=\"openai/gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}],
    input_cost_per_token=0.00001,
    output_cost_per_token=0.00003,
)
", Para [Str "[", Str "1", Str "]", Str "[", Str "4", Str "]"], Para [Strong [Str "Load Balancing"], Str ": The Router distributes traffic across multiple deployments of the same model using configurable strategies: simple-shuffle (default), usage-based, latency-based, least-busy, and cost-based routing. ", Str "[", Str "3", Str "]"], Para [Strong [Str "Virtual Keys and Spend Caps"], Str ": The Proxy Server issues virtual API keys with per-key budgets, rate limits (RPM, TPM), and concurrent request limits. Spend is tracked automatically per key, user, and team:"], CodeBlock ("", ["bash"], []) "curl -X POST http://0.0.0.0:4000/key/generate \\
  -H \"Authorization: Bearer sk-master-key\" \\
  -H \"Content-Type: application/json\" \\
  -d '{\"max_budget\": 100, \"tpm_limit\": 10000, \"rpm_limit\": 100}'
", Para [Str "[", Str "5", Str "]"], Para [Strong [Str "Observability Integration"], Str ": Send telemetry to external platforms through declarative callbacks:"], CodeBlock ("", ["python"], []) "import litellm

litellm.success_callback = [\"langfuse\", \"helicone\", \"lunary\"]
litellm.failure_callback = [\"sentry\", \"langfuse\"]
", Para [Str "Supported integrations include Langfuse, Helicone, Lunary, LangSmith, MLflow, Traceloop, Arize, PromptLayer, PostHog, Sentry, and Slack. ", Str "[", Str "7", Str "]"], Para [Strong [Str "Exception Mapping"], Str ": Provider errors are mapped to OpenAI exception types for consistent error handling:"], CodeBlock ("", ["python"], []) "import litellm
import openai

try:
    response = litellm.completion(model=\"anthropic/claude-opus-4-6\", messages=[...])
except openai.AuthenticationError as e:
    print(f\"Auth failed on {e.llm_provider}: {e.message}\")
except openai.RateLimitError as e:
    should_retry = litellm._should_retry(e.status_code)
", Para [Str "[", Str "6", Str "]"], Header 2 ("use-cases", ["unnumbered", "unlisted"], []) [Str "Use Cases"], Para [Strong [Str "Multi-Provider Abstraction"], Str ": Applications that need to call multiple LLM providers without maintaining separate client libraries for each. A single ", Code ("", [], []) "completion()", Str " call works across OpenAI, Anthropic, Azure, Vertex AI, and dozens of other providers."], Para [Strong [Str "Production LLM Gateway"], Str ": Organizations deploying the Proxy Server as a centralized gateway for all LLM traffic. Teams authenticate with virtual keys, budgets enforce cost controls, and the admin dashboard provides visibility into usage patterns."], Para [Strong [Str "Failover and Reliability"], Str ": Systems that need automatic failover when a primary provider experiences outages. The Router's cooldown and fallback mechanisms route traffic to healthy deployments without application-level changes."], Para [Strong [Str "Cost Optimization"], Str ": Teams routing traffic to the cheapest available deployment using cost-based routing, or using model aliasing to redirect expensive model requests to more affordable alternatives without changing client code."], Para [Strong [Str "Multi-Tenant Platforms"], Str ": SaaS applications that issue virtual keys to customers, each with independent spend caps and rate limits. The Proxy Server tracks per-tenant costs and enforces budgets automatically."], Para [Strong [Str "Development and Testing"], Str ": Developers using the SDK to test prompts across multiple providers during development, comparing response quality and latency before committing to a production provider."], Header 2 ("api-reference-summary", ["unnumbered", "unlisted"], []) [Str "API Reference Summary"], Header 3 ("sdk-functions", ["unnumbered", "unlisted"], []) [Str "SDK Functions"], BulletList [[Plain [Code ("", [], []) "completion(model, messages, **kwargs)", Str " -- Chat completion across all providers"]], [Plain [Code ("", [], []) "embedding(model, input, **kwargs)", Str " -- Text embedding generation"]], [Plain [Code ("", [], []) "image_generation(model, prompt, **kwargs)", Str " -- Image generation"]], [Plain [Code ("", [], []) "text_completion(model, prompt, **kwargs)", Str " -- Legacy text completion"]], [Plain [Code ("", [], []) "completion_cost(response)", Str " -- Calculate cost from a completion response"]]], Header 3 ("proxy-endpoints", ["unnumbered", "unlisted"], []) [Str "Proxy Endpoints"], BulletList [[Plain [Code ("", [], []) "POST /chat/completions", Str " -- OpenAI-compatible chat completion"]], [Plain [Code ("", [], []) "POST /completions", Str " -- Legacy text completion"]], [Plain [Code ("", [], []) "POST /embeddings", Str " -- Embedding generation"]], [Plain [Code ("", [], []) "POST /images/generations", Str " -- Image generation"]], [Plain [Code ("", [], []) "POST /audio/transcriptions", Str " -- Audio transcription"]], [Plain [Code ("", [], []) "POST /audio/speech", Str " -- Text-to-speech"]], [Plain [Code ("", [], []) "POST /batches", Str " -- Batch processing"]], [Plain [Code ("", [], []) "GET /models", Str " -- List available models"]], [Plain [Code ("", [], []) "POST /key/generate", Str " -- Create virtual key"]], [Plain [Code ("", [], []) "POST /key/info", Str " -- Get key spend and metadata"]], [Plain [Code ("", [], []) "POST /key/block", Str " -- Disable a virtual key"]], [Plain [Code ("", [], []) "POST /key/unblock", Str " -- Re-enable a virtual key"]], [Plain [Code ("", [], []) "POST /user/info", Str " -- Get user-level spend"]], [Plain [Code ("", [], []) "POST /team/info", Str " -- Get team-level spend"]], [Plain [Code ("", [], []) "GET /utils/transform_request", Str " -- Inspect request transformation"]]], Header 3 ("completion-parameters", ["unnumbered", "unlisted"], []) [Str "Completion Parameters"], BulletList [[Plain [Code ("", [], []) "model", Str " -- Provider-prefixed model ID (e.g., ", Code ("", [], []) "openai/gpt-4o", Str ")"]], [Plain [Code ("", [], []) "messages", Str " -- Conversation messages array (system, user, assistant, tool roles)"]], [Plain [Code ("", [], []) "temperature", Str " -- Sampling temperature"]], [Plain [Code ("", [], []) "max_tokens", Str " / ", Code ("", [], []) "max_completion_tokens", Str " -- Output token limit"]], [Plain [Code ("", [], []) "top_p", Str " -- Nucleus sampling threshold"]], [Plain [Code ("", [], []) "stream", Str " -- Enable streaming responses"]], [Plain [Code ("", [], []) "tools", Str " -- Tool/function definitions array"]], [Plain [Code ("", [], []) "tool_choice", Str " -- Tool selection control"]], [Plain [Code ("", [], []) "response_format", Str " -- JSON mode or structured output schema"]], [Plain [Code ("", [], []) "stop", Str " -- Stop sequences"]], [Plain [Code ("", [], []) "seed", Str " -- Deterministic output seed"]], [Plain [Code ("", [], []) "num_retries", Str " -- Automatic retry count"]], [Plain [Code ("", [], []) "fallbacks", Str " -- Fallback model list"]], [Plain [Code ("", [], []) "api_base", Str " -- Custom provider endpoint"]], [Plain [Code ("", [], []) "api_key", Str " -- Provider API key override"]], [Plain [Code ("", [], []) "metadata", Str " -- Custom metadata for logging"]], [Plain [Code ("", [], []) "input_cost_per_token", Str " -- Custom input pricing"]], [Plain [Code ("", [], []) "output_cost_per_token", Str " -- Custom output pricing", SoftBreak, Str "[", Str "1", Str "]", Str "[", Str "4", Str "]"]]], Header 2 ("configuration-and-customization", ["unnumbered", "unlisted"], []) [Str "Configuration and Customization"], Header 3 ("proxy-configuration-yaml", ["unnumbered", "unlisted"], []) [Str "Proxy Configuration (YAML)"], Para [Str "The Proxy Server is configured through a YAML file with four main sections:"], CodeBlock ("", ["yaml"], []) "model_list:
  - model_name: gpt-4o                    # User-facing alias
    litellm_params:
      model: openai/gpt-4o               # Actual provider model
      api_key: os.environ/OPENAI_API_KEY  # Environment variable reference
      rpm: 1000                           # Requests per minute limit
      tpm: 100000                         # Tokens per minute limit

  - model_name: gpt-4o                    # Second deployment (load balanced)
    litellm_params:
      model: azure/gpt-4o-eu
      api_base: https://my-azure.openai.azure.com
      api_key: os.environ/AZURE_API_KEY

router_settings:
  routing_strategy: simple-shuffle        # Load balancing strategy
  num_retries: 3                          # Retry attempts
  retry_after: 1                          # Minimum wait between retries (seconds)

litellm_settings:
  drop_params: true                       # Drop unsupported params silently
  set_verbose: false                      # Disable verbose logging

general_settings:
  master_key: sk-master-key               # Admin authentication key
  database_url: os.environ/DATABASE_URL   # PostgreSQL for spend tracking
  database_connection_pool_limit: 15      # Connections per worker
", Para [Str "Launch with: ", Code ("", [], []) "litellm --config /path/to/config.yaml", Str " ", Str "[", Str "8", Str "]"], Header 3 ("environment-variable-loading", ["unnumbered", "unlisted"], []) [Str "Environment Variable Loading"], Para [Str "Configuration values prefixed with ", Code ("", [], []) "os.environ/", Str " are resolved from environment variables at startup, keeping secrets out of configuration files. ", Str "[", Str "8", Str "]"], Header 3 ("credential-lists", ["unnumbered", "unlisted"], []) [Str "Credential Lists"], Para [Str "Define credentials once and reference them across multiple models:"], CodeBlock ("", ["yaml"], []) "credential_list:
  - credential_name: azure-prod
    api_key: os.environ/AZURE_PROD_KEY
    api_base: https://prod.openai.azure.com

model_list:
  - model_name: gpt-4o
    litellm_params:
      model: azure/gpt-4o
      litellm_credential_name: azure-prod
", Para [Str "[", Str "8", Str "]"], Header 3 ("wildcard-models", ["unnumbered", "unlisted"], []) [Str "Wildcard Models"], Para [Str "Route any model through default credentials using wildcards:"], CodeBlock ("", ["yaml"], []) "model_list:
  - model_name: \"*\"
    litellm_params:
      model: \"*\"
", Para [Str "[", Str "8", Str "]"], Header 3 ("router-strategies", ["unnumbered", "unlisted"], []) [Str "Router Strategies"], BulletList [[Plain [Code ("", [], []) "simple-shuffle", Str " -- Default. Random selection weighted by RPM/TPM limits. Lowest latency overhead"]], [Plain [Code ("", [], []) "usage-based-routing-v2", Str " -- Routes to deployments with lowest TPM usage (requires Redis)"]], [Plain [Code ("", [], []) "latency-based-routing", Str " -- Selects deployment with lowest observed response time"]], [Plain [Code ("", [], []) "least-busy", Str " -- Routes to deployment with fewest active requests"]], [Plain [Code ("", [], []) "cost-based-routing", Str " -- Selects cheapest available deployment", SoftBreak, Str "[", Str "3", Str "]"]]], Header 3 ("cooldown-configuration", ["unnumbered", "unlisted"], []) [Str "Cooldown Configuration"], Para [Str "Deployments experiencing failures are automatically cooled down:"], CodeBlock ("", ["yaml"], []) "router_settings:
  allowed_fails: 3              # Failures before cooldown triggers
  cooldown_time: 5              # Cooldown duration in seconds
", Para [Str "[", Str "3", Str "]"], Header 2 ("integration-patterns", ["unnumbered", "unlisted"], []) [Str "Integration Patterns"], Header 3 ("with-openai-sdk", ["unnumbered", "unlisted"], []) [Str "With OpenAI SDK"], Para [Str "Point any OpenAI SDK client at the Proxy Server:"], CodeBlock ("", ["python"], []) "from openai import OpenAI

client = OpenAI(base_url=\"http://0.0.0.0:4000\", api_key=\"sk-virtual-key\")
response = client.chat.completions.create(
    model=\"gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}]
)
", Para [Str "[", Str "2", Str "]"], Header 3 ("with-langchain", ["unnumbered", "unlisted"], []) [Str "With LangChain"], CodeBlock ("", ["python"], []) "from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model=\"gpt-4o\",
    openai_api_base=\"http://0.0.0.0:4000\",
    openai_api_key=\"sk-virtual-key\",
)
", Para [Str "[", Str "2", Str "]"], Header 3 ("with-observability-platforms", ["unnumbered", "unlisted"], []) [Str "With Observability Platforms"], Para [Str "SDK-level callbacks send telemetry without proxy overhead:"], CodeBlock ("", ["python"], []) "import litellm
import os

os.environ[\"LANGFUSE_PUBLIC_KEY\"] = \"pk-...\"
os.environ[\"LANGFUSE_SECRET_KEY\"] = \"sk-...\"

litellm.success_callback = [\"langfuse\"]
litellm.failure_callback = [\"langfuse\"]

# All subsequent completion calls are automatically traced
response = litellm.completion(model=\"openai/gpt-4o\", messages=[...])
", Para [Str "[", Str "7", Str "]"], Header 3 ("with-agent-frameworks", ["unnumbered", "unlisted"], []) [Str "With Agent Frameworks"], Para [Str "LiteLLM can serve as the LLM backend for agent frameworks by running the Proxy Server and pointing the framework's OpenAI client at the proxy URL. This centralizes provider credentials, adds cost tracking, and enables model routing without modifying the framework's code."], Header 3 ("with-docker-and-kubernetes", ["unnumbered", "unlisted"], []) [Str "With Docker and Kubernetes"], Para [Str "Deploy the Proxy Server as a container with configuration mounted as a volume. Helm charts and Terraform modules are available for Kubernetes deployments. The proxy handles 1,500+ requests per second under load testing. ", Str "[", Str "2", Str "]"], Header 2 ("examples", ["unnumbered", "unlisted"], []) [Str "Examples"], Header 3 ("multi-provider-completion", ["unnumbered", "unlisted"], []) [Str "Multi-Provider Completion"], CodeBlock ("", ["python"], []) "from litellm import completion

# Same interface for every provider
providers = [
    \"openai/gpt-4o\",
    \"anthropic/claude-sonnet-4-6\",
    \"azure/gpt-4o-eu\",
    \"ollama/llama3\",
]

for model in providers:
    response = completion(
        model=model,
        messages=[{\"role\": \"user\", \"content\": \"What is 2+2?\"}]
    )
    print(f\"{model}: {response.choices[0].message.content}\")
", Header 3 ("router-with-fallbacks", ["unnumbered", "unlisted"], []) [Str "Router with Fallbacks"], CodeBlock ("", ["python"], []) "from litellm import Router

router = Router(
    model_list=[
        {
            \"model_name\": \"gpt-4o\",
            \"litellm_params\": {\"model\": \"openai/gpt-4o\", \"api_key\": \"sk-...\"},
            \"rpm\": 500,
        },
        {
            \"model_name\": \"gpt-4o\",
            \"litellm_params\": {\"model\": \"azure/gpt-4o-eu\", \"api_key\": \"az-...\"},
            \"rpm\": 1000,
        },
    ],
    routing_strategy=\"simple-shuffle\",
    num_retries=3,
)

response = router.completion(
    model=\"gpt-4o\",
    messages=[{\"role\": \"user\", \"content\": \"Hello\"}]
)
", Para [Str "[", Str "3", Str "]"], Header 3 ("proxy-with-virtual-keys-and-budgets", ["unnumbered", "unlisted"], []) [Str "Proxy with Virtual Keys and Budgets"], CodeBlock ("", ["yaml"], []) "# config.yaml
model_list:
  - model_name: gpt-4o
    litellm_params:
      model: openai/gpt-4o
      api_key: os.environ/OPENAI_API_KEY

general_settings:
  master_key: sk-master-key
  database_url: os.environ/DATABASE_URL
", CodeBlock ("", ["bash"], []) "# Start proxy
litellm --config config.yaml

# Generate a virtual key with $50 budget and rate limits
curl -X POST http://0.0.0.0:4000/key/generate \\
  -H \"Authorization: Bearer sk-master-key\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"max_budget\": 50,
    \"rpm_limit\": 100,
    \"tpm_limit\": 50000,
    \"budget_duration\": \"30d\"
  }'

# Client uses the virtual key
curl -X POST http://0.0.0.0:4000/chat/completions \\
  -H \"Authorization: Bearer sk-generated-virtual-key\" \\
  -H \"Content-Type: application/json\" \\
  -d '{
    \"model\": \"gpt-4o\",
    \"messages\": [{\"role\": \"user\", \"content\": \"Hello\"}]
  }'
", Para [Str "[", Str "2", Str "]", Str "[", Str "5", Str "]"], Header 3 ("streaming-with-cost-tracking", ["unnumbered", "unlisted"], []) [Str "Streaming with Cost Tracking"], CodeBlock ("", ["python"], []) "import litellm

litellm.success_callback = [\"langfuse\"]

response = litellm.completion(
    model=\"anthropic/claude-sonnet-4-6\",
    messages=[{\"role\": \"user\", \"content\": \"Explain quantum computing\"}],
    stream=True,
)

for chunk in response:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end=\"\")

# Cost is automatically calculated and sent to Langfuse
", Para [Str "[", Str "1", Str "]", Str "[", Str "7", Str "]"], Header 2 ("limitations-and-considerations", ["unnumbered", "unlisted"], []) [Str "Limitations and Considerations"], BulletList [[Plain [Strong [Str "Python-only SDK"], Str ": The SDK is Python-only. Non-Python applications must use the Proxy Server and connect via HTTP with an OpenAI-compatible client library"]], [Plain [Strong [Str "Provider parameter coverage"], Str ": Not all provider-specific parameters are supported through the unified interface. The ", Code ("", [], []) "drop_params", Str " setting silently drops unsupported parameters rather than raising errors"]], [Plain [Strong [Str "PostgreSQL requirement for spend tracking"], Str ": Virtual key management and spend tracking require a PostgreSQL database connection on the Proxy Server"]], [Plain [Strong [Str "Redis for advanced routing"], Str ": Usage-based and latency-based routing strategies require a Redis instance for cross-process metric sharing"]], [Plain [Strong [Str "Model pricing accuracy"], Str ": Built-in cost calculations depend on LiteLLM's pricing data, which may lag behind provider pricing changes. Custom per-token pricing can override defaults"]], [Plain [Strong [Str "Exception mapping coverage"], Str ": Not all providers support the full set of mapped exception types. OpenAI and Anthropic have comprehensive mapping; smaller providers may have limited coverage ", Str "[", Str "6", Str "]"]], [Plain [Strong [Str "Streaming error propagation"], Str ": Errors during streaming propagate as exceptions during chunk iteration, which requires error handling within the streaming loop"]], [Plain [Strong [Str "Proxy latency overhead"], Str ": The Proxy Server adds network hop latency compared to direct SDK calls. For latency-sensitive applications, the SDK with in-process Router may be preferable"]]], Header 2 ("changelog-highlights", ["unnumbered", "unlisted"], []) [Str "Changelog Highlights"], BulletList [[Plain [Strong [Str "100+ provider support"], Str ": Expanded from initial providers to support over 100 LLM APIs through the OpenAI format"]], [Plain [Strong [Str "Proxy Server (LLM Gateway)"], Str ": Standalone HTTP server with authentication, virtual keys, and admin dashboard"]], [Plain [Strong [Str "Router load balancing"], Str ": Multiple routing strategies (simple-shuffle, usage-based, latency-based, least-busy, cost-based)"]], [Plain [Strong [Str "Virtual key management"], Str ": Per-key budgets, rate limits, and team-based spend tracking with PostgreSQL backend"]], [Plain [Strong [Str "Observability callbacks"], Str ": Integration with Langfuse, Helicone, Lunary, LangSmith, MLflow, Arize, and others"]], [Plain [Strong [Str "Credential lists"], Str ": Centralized credential management with reference-based model configuration"]], [Plain [Strong [Str "Custom routing strategies"], Str ": Extensible routing through ", Code ("", [], []) "CustomRoutingStrategyBase", Str " for deployment selection logic"]], [Plain [Strong [Str "Enterprise features"], Str ": Automatic key rotation, audit logging, and granular access controls"]], [Plain [Strong [Str "Performance"], Str ": 1,500+ requests per second throughput under load testing"]]], Header 2 ("citations", ["unnumbered", "unlisted"], []) [Str "Citations"], BulletList [[Plain [Str "[", Str "1", Str "]", Str " LiteLLM Documentation - ", Link ("", [], []) [Str "https://docs.litellm.ai/docs/"] ("https://docs.litellm.ai/docs/", "")]], [Plain [Str "[", Str "2", Str "]", Str " Proxy Quick Start - ", Link ("", [], []) [Str "https://docs.litellm.ai/docs/proxy/quick_start"] ("https://docs.litellm.ai/docs/proxy/quick_start", "")]], [Plain [Str "[", Str "3", Str "]", Str " Router Documentation - ", Link ("", [], []) [Str "https://docs.litellm.ai/docs/routing"] ("https://docs.litellm.ai/docs/routing", "")]], [Plain [Str "[", Str "4", Str "]", Str " Completion Input Parameters - ", Link ("", [], []) [Str "https://docs.litellm.ai/docs/completion/input"] ("https://docs.litellm.ai/docs/completion/input", "")]], [Plain [Str "[", Str "5", Str "]", Str " Virtual Keys - ", Link ("", [], []) [Str "https://docs.litellm.ai/docs/proxy/virtual_keys"] ("https://docs.litellm.ai/docs/proxy/virtual_keys", "")]], [Plain [Str "[", Str "6", Str "]", Str " Exception Mapping - ", Link ("", [], []) [Str "https://docs.litellm.ai/docs/exception_mapping"] ("https://docs.litellm.ai/docs/exception_mapping", "")]], [Plain [Str "[", Str "7", Str "]", Str " Observability Callbacks - ", Link ("", [], []) [Str "https://docs.litellm.ai/docs/observability/callbacks"] ("https://docs.litellm.ai/docs/observability/callbacks", "")]], [Plain [Str "[", Str "8", Str "]", Str " Proxy Configuration - ", Link ("", [], []) [Str "https://docs.litellm.ai/docs/proxy/configs"] ("https://docs.litellm.ai/docs/proxy/configs", "")]]], Header 2 ("discovery-signals", ["unnumbered", "unlisted"], []) [Str "Discovery Signals"], BlockQuote [Para [Str "Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search."]], Header 3 ("keywords", ["unnumbered", "unlisted"], []) [Str "Keywords"], BulletList [[Plain [Str "litellm"]], [Plain [Str "BerriAI"]], [Plain [Str "llm gateway"]], [Plain [Str "model gateway"]], [Plain [Str "unified llm api"]], [Plain [Str "openai-compatible proxy"]], [Plain [Str "completion()"]], [Plain [Str "provider prefix"]], [Plain [Str "router"]], [Plain [Str "virtual keys"]], [Plain [Str "spend tracking"]], [Plain [Str "fallback routing"]], [Plain [Str "exception mapping"]], [Plain [Str "cost-based routing"]], [Plain [Str "latency-based routing"]], [Plain [Str "usage-based routing"]], [Plain [Str "least-busy routing"]], [Plain [Str "simple-shuffle"]], [Plain [Str "cooldown"]], [Plain [Str "multi-tenant llm"]], [Plain [Str "master key"]], [Plain [Str "proxy server"]], [Plain [Str "credential list"]], [Plain [Str "wildcard model"]], [Plain [Str "langfuse callback"]], [Plain [Str "helicone callback"]], [Plain [Str "drop_params"]], [Plain [Str "100+ providers"]]], Header 3 ("verb-noun-tasks", ["unnumbered", "unlisted"], []) [Str "Verb-Noun Tasks"], BulletList [[Plain [Str "Call OpenAI, Anthropic, Azure, and Vertex AI through one ", Code ("", [], []) "completion()", Str " function"]], [Plain [Str "Map provider exceptions to OpenAI exception types"]], [Plain [Str "Configure retry-and-fallback chains across multiple providers"]], [Plain [Str "Issue virtual API keys with per-key budgets and rate limits"]], [Plain [Str "Route traffic across deployments using cost, latency, or usage strategies"]], [Plain [Str "Track per-team and per-user spend in PostgreSQL"]], [Plain [Str "Push request telemetry to Langfuse, Helicone, LangSmith, or Arize"]], [Plain [Str "Deploy a centralized LLM gateway behind an OpenAI-compatible URL"]], [Plain [Str "Define credential lists and reference them across model entries"]], [Plain [Str "Cool down failing deployments automatically"]], [Plain [Str "Override per-token pricing for a custom deployment"]], [Plain [Str "Run the proxy in Docker or Kubernetes with Helm"]]], Header 3 ("user-intent-phrases", ["unnumbered", "unlisted"], []) [Str "User Intent Phrases"], BulletList [[Plain [Str "How do I call Anthropic and OpenAI with the same code?"]], [Plain [Str "I want one Python function that works across every LLM provider."]], [Plain [Str "How do I add automatic failover when OpenAI is down?"]], [Plain [Str "How do I give each team in my company a separate LLM budget?"]], [Plain [Str "How do I issue API keys with spend caps?"]], [Plain [Str "I need to centralize LLM credentials so apps never see raw provider keys."]], [Plain [Str "How do I track LLM costs per user?"]], [Plain [Str "Can I route the cheapest available model automatically?"]], [Plain [Str "How do I add Langfuse tracing to every LLM call without changing my agent code?"]], [Plain [Str "How do I run an OpenAI-compatible gateway in front of 10 different providers?"]], [Plain [Str "How do I load-balance across multiple Azure OpenAI deployments?"]], [Plain [Str "I want exponential backoff and retries that work the same way across providers."]]], Header 3 ("problem-statements", ["unnumbered", "unlisted"], []) [Str "Problem Statements"], BulletList [[Plain [Str "Each LLM provider has a different SDK and error model and my code is full of branches."]], [Plain [Str "A single provider outage takes down my product."]], [Plain [Str "Engineers are leaking provider API keys into repos because there is no central gateway."]], [Plain [Str "I cannot tell which team is spending how much on LLM tokens."]], [Plain [Str "I have multiple Azure deployments and no way to load-balance them."]], [Plain [Str "I am paying for premium models when a cheaper one would have answered."]], [Plain [Str "Observability tools require per-provider integration work I keep redoing."]]], Header 3 ("when-to-pick-this", ["unnumbered", "unlisted"], []) [Str "When to Pick This"], BulletList [[Plain [Str "Pick this when you want an open-source, developer-driven gateway with a Python SDK you can embed (vs Portkey's managed-first model)."]], [Plain [Str "Pick this when you need cost-based, latency-based, and usage-based routing strategies out of the box."]], [Plain [Str "Pick this when you want virtual keys with PostgreSQL-backed spend tracking you self-host."]], [Plain [Str "Pick this when most of your stack is Python and a single in-process function is preferable to a network hop."]], [Plain [Str "Pick this when you need text generation, embeddings, and image generation, but not music or video (vs ccapi)."]], [Plain [Str "Pick this when you want declarative observability callbacks instead of building each integration yourself."]], [Plain [Str "Pick this when 100+ providers and broad community coverage matter more than enterprise routing primitives."]]], Header 3 ("related-terms-and-aliases", ["unnumbered", "unlisted"], []) [Str "Related Terms and Aliases"], BulletList [[Plain [Str "llm proxy"]], [Plain [Str "llm router"]], [Plain [Str "multi-provider sdk"]], [Plain [Str "model abstraction layer"]], [Plain [Str "openai shim"]], [Plain [Str "llm cost tracking"]], [Plain [Str "llm spend management"]], [Plain [Str "failover gateway"]], [Plain [Str "llm load balancer"]], [Plain [Str "BerriAI proxy"]]]]