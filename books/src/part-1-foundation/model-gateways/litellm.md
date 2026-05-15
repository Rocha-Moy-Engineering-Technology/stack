# LiteLLM

> Python SDK and proxy for unified access to 100+ LLM APIs

| Field | Value |
|-------|-------|
| Group | Model Gateways |
| Type | SDK |
| Open Source | Yes |
| GitHub | [BerriAI/litellm](https://github.com/BerriAI/litellm) |
| Stars | 46980 |
| Documentation | [Official Docs](https://docs.litellm.ai/docs/) |

## Overview

LiteLLM is an open source Python SDK and proxy server that provides a unified interface for calling 100+ large language models using the OpenAI input/output format. It translates inputs to provider-specific endpoints and normalizes responses into a consistent format, allowing developers to switch between providers without rewriting application code. The project is maintained by BerriAI and supports providers including OpenAI, Anthropic, xAI, Google Vertex AI, Azure OpenAI, NVIDIA NIM, HuggingFace, Ollama, OpenRouter, Novita AI, and Vercel AI Gateway. [1]

LiteLLM ships as two components. The **Python SDK** embeds directly into applications and provides completion calls, retry/fallback logic, observability callbacks, and cost tracking. The **Proxy Server** (also called the LLM Gateway) runs as a standalone service that exposes an OpenAI-compatible API with authentication, multi-tenant cost tracking, virtual keys, rate limiting, load balancing, and an admin dashboard. Both components share the same underlying translation layer and provider support. [1]

## Core Concepts

**Unified Completion Interface**: The `completion()` function accepts an OpenAI-style model identifier and messages array, translating the request to the target provider's native format. Responses are normalized to match OpenAI's chat completion structure regardless of the underlying provider. [1]

**Provider Prefixes**: Models are specified using a `provider/model-name` format (e.g., `openai/gpt-4o`, `anthropic/claude-opus-4-6`, `azure/gpt-4o-eu`). The prefix tells LiteLLM which translation layer to apply for the request and response. [1][4]

**Router**: The Router manages load balancing across multiple deployments of the same model. Deployments sharing the same `model_name` form a model group, and the Router selects among them using configurable strategies. The Router also handles cooldowns, retries, and fallbacks when deployments fail. [3]

**Virtual Keys**: The Proxy Server issues virtual API keys that abstract over underlying provider credentials. Each virtual key can have its own spend budget, rate limits, and model access restrictions. Users and teams interact with virtual keys rather than raw provider API keys. [5]

**Exception Mapping**: Provider-specific errors are mapped to OpenAI exception types (`AuthenticationError`, `RateLimitError`, `APIError`, `Timeout`, `NotFoundError`, `ServiceUnavailableError`, `ContentPolicyViolationError`). All exceptions include `status_code`, `message`, and `llm_provider` attributes for debugging. [6]

**Observability Callbacks**: Three callback types (input, success, failure) send telemetry to external platforms. Callbacks are configured declaratively by setting `litellm.success_callback` and `litellm.failure_callback` to lists of integration names. [7]

## Architecture

LiteLLM is organized into three layers:

```
SDK Layer              completion(), embedding(), image_generation()
    |                  Provider translation, response normalization
    |
Router Layer           Load balancing, retries, fallbacks, cooldowns
    |                  Model groups, deployment health tracking
    |
Proxy Layer            HTTP server, virtual keys, spend tracking
                       Admin dashboard, rate limiting, auth
```

**SDK Layer**: The core translation engine. Each provider has a handler that converts OpenAI-format requests into provider-native API calls and normalizes responses back to OpenAI format. The SDK is stateless and can be embedded directly into Python applications. [1]

**Router Layer**: Sits on top of the SDK and manages multiple deployments. It tracks deployment health, enforces rate limits (Requests Per Minute (RPM) and Tokens Per Minute (TPM)), and applies routing strategies to distribute traffic. The Router operates in-process for SDK usage or as part of the Proxy Server. [3]

**Proxy Layer**: A standalone HTTP server built on the Router. It adds authentication (master key and virtual keys), a PostgreSQL-backed spend tracking database, team and user management, and an admin dashboard. Clients interact with it using standard OpenAI SDKs pointed at the proxy's base URL. [2][5]

## Key Features and Functionality

**Unified Completion Calls**: Call any supported provider through a single function with consistent input/output format:

```python
from litellm import completion

# OpenAI
response = completion(model="openai/gpt-4o", messages=[{"role": "user", "content": "Hi"}])

# Anthropic
response = completion(model="anthropic/claude-opus-4-6", messages=[{"role": "user", "content": "Hi"}])

# Azure OpenAI
response = completion(model="azure/gpt-4o-eu", messages=[{"role": "user", "content": "Hi"}])
```
[1]

**Streaming**: All providers support streaming via `stream=True`. Streamed responses return chunks in OpenAI's Server-Sent Events (SSE) format with token-level deltas and usage metadata:

```python
response = completion(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Write a story"}],
    stream=True,
)
for chunk in response:
    print(chunk.choices[0].delta.content or "", end="")
```
[1]

**Retry and Fallback Logic**: Configure automatic retries with exponential backoff and model fallbacks:

```python
from litellm import completion

response = completion(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
    num_retries=3,
    fallbacks=["anthropic/claude-sonnet-4-6", "azure/gpt-4o"],
)
```
[1]

**Cost Tracking**: LiteLLM calculates costs per request using built-in model pricing data. Custom per-token pricing can be specified per deployment:

```python
response = completion(
    model="openai/gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
    input_cost_per_token=0.00001,
    output_cost_per_token=0.00003,
)
```
[1][4]

**Load Balancing**: The Router distributes traffic across multiple deployments of the same model using configurable strategies: simple-shuffle (default), usage-based, latency-based, least-busy, and cost-based routing. [3]

**Virtual Keys and Spend Caps**: The Proxy Server issues virtual API keys with per-key budgets, rate limits (RPM, TPM), and concurrent request limits. Spend is tracked automatically per key, user, and team:

```bash
curl -X POST http://0.0.0.0:4000/key/generate \
  -H "Authorization: Bearer sk-master-key" \
  -H "Content-Type: application/json" \
  -d '{"max_budget": 100, "tpm_limit": 10000, "rpm_limit": 100}'
```
[5]

**Observability Integration**: Send telemetry to external platforms through declarative callbacks:

```python
import litellm

litellm.success_callback = ["langfuse", "helicone", "lunary"]
litellm.failure_callback = ["sentry", "langfuse"]
```

Supported integrations include Langfuse, Helicone, Lunary, LangSmith, MLflow, Traceloop, Arize, PromptLayer, PostHog, Sentry, and Slack. [7]

**Exception Mapping**: Provider errors are mapped to OpenAI exception types for consistent error handling:

```python
import litellm
import openai

try:
    response = litellm.completion(model="anthropic/claude-opus-4-6", messages=[...])
except openai.AuthenticationError as e:
    print(f"Auth failed on {e.llm_provider}: {e.message}")
except openai.RateLimitError as e:
    should_retry = litellm._should_retry(e.status_code)
```
[6]

## Use Cases

**Multi-Provider Abstraction**: Applications that need to call multiple LLM providers without maintaining separate client libraries for each. A single `completion()` call works across OpenAI, Anthropic, Azure, Vertex AI, and dozens of other providers.

**Production LLM Gateway**: Organizations deploying the Proxy Server as a centralized gateway for all LLM traffic. Teams authenticate with virtual keys, budgets enforce cost controls, and the admin dashboard provides visibility into usage patterns.

**Failover and Reliability**: Systems that need automatic failover when a primary provider experiences outages. The Router's cooldown and fallback mechanisms route traffic to healthy deployments without application-level changes.

**Cost Optimization**: Teams routing traffic to the cheapest available deployment using cost-based routing, or using model aliasing to redirect expensive model requests to more affordable alternatives without changing client code.

**Multi-Tenant Platforms**: SaaS applications that issue virtual keys to customers, each with independent spend caps and rate limits. The Proxy Server tracks per-tenant costs and enforces budgets automatically.

**Development and Testing**: Developers using the SDK to test prompts across multiple providers during development, comparing response quality and latency before committing to a production provider.

## API Reference Summary

### SDK Functions

- `completion(model, messages, **kwargs)` -- Chat completion across all providers
- `embedding(model, input, **kwargs)` -- Text embedding generation
- `image_generation(model, prompt, **kwargs)` -- Image generation
- `text_completion(model, prompt, **kwargs)` -- Legacy text completion
- `completion_cost(response)` -- Calculate cost from a completion response

### Proxy Endpoints

- `POST /chat/completions` -- OpenAI-compatible chat completion
- `POST /completions` -- Legacy text completion
- `POST /embeddings` -- Embedding generation
- `POST /images/generations` -- Image generation
- `POST /audio/transcriptions` -- Audio transcription
- `POST /audio/speech` -- Text-to-speech
- `POST /batches` -- Batch processing
- `GET /models` -- List available models
- `POST /key/generate` -- Create virtual key
- `POST /key/info` -- Get key spend and metadata
- `POST /key/block` -- Disable a virtual key
- `POST /key/unblock` -- Re-enable a virtual key
- `POST /user/info` -- Get user-level spend
- `POST /team/info` -- Get team-level spend
- `GET /utils/transform_request` -- Inspect request transformation

### Completion Parameters

- `model` -- Provider-prefixed model ID (e.g., `openai/gpt-4o`)
- `messages` -- Conversation messages array (system, user, assistant, tool roles)
- `temperature` -- Sampling temperature
- `max_tokens` / `max_completion_tokens` -- Output token limit
- `top_p` -- Nucleus sampling threshold
- `stream` -- Enable streaming responses
- `tools` -- Tool/function definitions array
- `tool_choice` -- Tool selection control
- `response_format` -- JSON mode or structured output schema
- `stop` -- Stop sequences
- `seed` -- Deterministic output seed
- `num_retries` -- Automatic retry count
- `fallbacks` -- Fallback model list
- `api_base` -- Custom provider endpoint
- `api_key` -- Provider API key override
- `metadata` -- Custom metadata for logging
- `input_cost_per_token` -- Custom input pricing
- `output_cost_per_token` -- Custom output pricing
[1][4]

## Configuration and Customization

### Proxy Configuration (YAML)

The Proxy Server is configured through a YAML file with four main sections:

```yaml
model_list:
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
```

Launch with: `litellm --config /path/to/config.yaml` [8]

### Environment Variable Loading

Configuration values prefixed with `os.environ/` are resolved from environment variables at startup, keeping secrets out of configuration files. [8]

### Credential Lists

Define credentials once and reference them across multiple models:

```yaml
credential_list:
  - credential_name: azure-prod
    api_key: os.environ/AZURE_PROD_KEY
    api_base: https://prod.openai.azure.com

model_list:
  - model_name: gpt-4o
    litellm_params:
      model: azure/gpt-4o
      litellm_credential_name: azure-prod
```
[8]

### Wildcard Models

Route any model through default credentials using wildcards:

```yaml
model_list:
  - model_name: "*"
    litellm_params:
      model: "*"
```
[8]

### Router Strategies

- `simple-shuffle` -- Default. Random selection weighted by RPM/TPM limits. Lowest latency overhead
- `usage-based-routing-v2` -- Routes to deployments with lowest TPM usage (requires Redis)
- `latency-based-routing` -- Selects deployment with lowest observed response time
- `least-busy` -- Routes to deployment with fewest active requests
- `cost-based-routing` -- Selects cheapest available deployment
[3]

### Cooldown Configuration

Deployments experiencing failures are automatically cooled down:

```yaml
router_settings:
  allowed_fails: 3              # Failures before cooldown triggers
  cooldown_time: 5              # Cooldown duration in seconds
```
[3]

## Integration Patterns

### With OpenAI SDK

Point any OpenAI SDK client at the Proxy Server:

```python
from openai import OpenAI

client = OpenAI(base_url="http://0.0.0.0:4000", api_key="sk-virtual-key")
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
```
[2]

### With LangChain

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4o",
    openai_api_base="http://0.0.0.0:4000",
    openai_api_key="sk-virtual-key",
)
```
[2]

### With Observability Platforms

SDK-level callbacks send telemetry without proxy overhead:

```python
import litellm
import os

os.environ["LANGFUSE_PUBLIC_KEY"] = "pk-..."
os.environ["LANGFUSE_SECRET_KEY"] = "sk-..."

litellm.success_callback = ["langfuse"]
litellm.failure_callback = ["langfuse"]

# All subsequent completion calls are automatically traced
response = litellm.completion(model="openai/gpt-4o", messages=[...])
```
[7]

### With Agent Frameworks

LiteLLM can serve as the LLM backend for agent frameworks by running the Proxy Server and pointing the framework's OpenAI client at the proxy URL. This centralizes provider credentials, adds cost tracking, and enables model routing without modifying the framework's code.

### With Docker and Kubernetes

Deploy the Proxy Server as a container with configuration mounted as a volume. Helm charts and Terraform modules are available for Kubernetes deployments. The proxy handles 1,500+ requests per second under load testing. [2]

## Examples

### Multi-Provider Completion

```python
from litellm import completion

# Same interface for every provider
providers = [
    "openai/gpt-4o",
    "anthropic/claude-sonnet-4-6",
    "azure/gpt-4o-eu",
    "ollama/llama3",
]

for model in providers:
    response = completion(
        model=model,
        messages=[{"role": "user", "content": "What is 2+2?"}]
    )
    print(f"{model}: {response.choices[0].message.content}")
```

### Router with Fallbacks

```python
from litellm import Router

router = Router(
    model_list=[
        {
            "model_name": "gpt-4o",
            "litellm_params": {"model": "openai/gpt-4o", "api_key": "sk-..."},
            "rpm": 500,
        },
        {
            "model_name": "gpt-4o",
            "litellm_params": {"model": "azure/gpt-4o-eu", "api_key": "az-..."},
            "rpm": 1000,
        },
    ],
    routing_strategy="simple-shuffle",
    num_retries=3,
)

response = router.completion(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
```
[3]

### Proxy with Virtual Keys and Budgets

```yaml
# config.yaml
model_list:
  - model_name: gpt-4o
    litellm_params:
      model: openai/gpt-4o
      api_key: os.environ/OPENAI_API_KEY

general_settings:
  master_key: sk-master-key
  database_url: os.environ/DATABASE_URL
```

```bash
# Start proxy
litellm --config config.yaml

# Generate a virtual key with $50 budget and rate limits
curl -X POST http://0.0.0.0:4000/key/generate \
  -H "Authorization: Bearer sk-master-key" \
  -H "Content-Type: application/json" \
  -d '{
    "max_budget": 50,
    "rpm_limit": 100,
    "tpm_limit": 50000,
    "budget_duration": "30d"
  }'

# Client uses the virtual key
curl -X POST http://0.0.0.0:4000/chat/completions \
  -H "Authorization: Bearer sk-generated-virtual-key" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```
[2][5]

### Streaming with Cost Tracking

```python
import litellm

litellm.success_callback = ["langfuse"]

response = litellm.completion(
    model="anthropic/claude-sonnet-4-6",
    messages=[{"role": "user", "content": "Explain quantum computing"}],
    stream=True,
)

for chunk in response:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end="")

# Cost is automatically calculated and sent to Langfuse
```
[1][7]

## Limitations and Considerations

- **Python-only SDK**: The SDK is Python-only. Non-Python applications must use the Proxy Server and connect via HTTP with an OpenAI-compatible client library
- **Provider parameter coverage**: Not all provider-specific parameters are supported through the unified interface. The `drop_params` setting silently drops unsupported parameters rather than raising errors
- **PostgreSQL requirement for spend tracking**: Virtual key management and spend tracking require a PostgreSQL database connection on the Proxy Server
- **Redis for advanced routing**: Usage-based and latency-based routing strategies require a Redis instance for cross-process metric sharing
- **Model pricing accuracy**: Built-in cost calculations depend on LiteLLM's pricing data, which may lag behind provider pricing changes. Custom per-token pricing can override defaults
- **Exception mapping coverage**: Not all providers support the full set of mapped exception types. OpenAI and Anthropic have comprehensive mapping; smaller providers may have limited coverage [6]
- **Streaming error propagation**: Errors during streaming propagate as exceptions during chunk iteration, which requires error handling within the streaming loop
- **Proxy latency overhead**: The Proxy Server adds network hop latency compared to direct SDK calls. For latency-sensitive applications, the SDK with in-process Router may be preferable

## Changelog Highlights

- **100+ provider support**: Expanded from initial providers to support over 100 LLM APIs through the OpenAI format
- **Proxy Server (LLM Gateway)**: Standalone HTTP server with authentication, virtual keys, and admin dashboard
- **Router load balancing**: Multiple routing strategies (simple-shuffle, usage-based, latency-based, least-busy, cost-based)
- **Virtual key management**: Per-key budgets, rate limits, and team-based spend tracking with PostgreSQL backend
- **Observability callbacks**: Integration with Langfuse, Helicone, Lunary, LangSmith, MLflow, Arize, and others
- **Credential lists**: Centralized credential management with reference-based model configuration
- **Custom routing strategies**: Extensible routing through `CustomRoutingStrategyBase` for deployment selection logic
- **Enterprise features**: Automatic key rotation, audit logging, and granular access controls
- **Performance**: 1,500+ requests per second throughput under load testing

## Citations

- [1] LiteLLM Documentation - <https://docs.litellm.ai/docs/>
- [2] Proxy Quick Start - <https://docs.litellm.ai/docs/proxy/quick_start>
- [3] Router Documentation - <https://docs.litellm.ai/docs/routing>
- [4] Completion Input Parameters - <https://docs.litellm.ai/docs/completion/input>
- [5] Virtual Keys - <https://docs.litellm.ai/docs/proxy/virtual_keys>
- [6] Exception Mapping - <https://docs.litellm.ai/docs/exception_mapping>
- [7] Observability Callbacks - <https://docs.litellm.ai/docs/observability/callbacks>
- [8] Proxy Configuration - <https://docs.litellm.ai/docs/proxy/configs>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- litellm
- BerriAI
- llm gateway
- model gateway
- unified llm api
- openai-compatible proxy
- completion()
- provider prefix
- router
- virtual keys
- spend tracking
- fallback routing
- exception mapping
- cost-based routing
- latency-based routing
- usage-based routing
- least-busy routing
- simple-shuffle
- cooldown
- multi-tenant llm
- master key
- proxy server
- credential list
- wildcard model
- langfuse callback
- helicone callback
- drop_params
- 100+ providers

### Verb-Noun Tasks

- Call OpenAI, Anthropic, Azure, and Vertex AI through one `completion()` function
- Map provider exceptions to OpenAI exception types
- Configure retry-and-fallback chains across multiple providers
- Issue virtual API keys with per-key budgets and rate limits
- Route traffic across deployments using cost, latency, or usage strategies
- Track per-team and per-user spend in PostgreSQL
- Push request telemetry to Langfuse, Helicone, LangSmith, or Arize
- Deploy a centralized LLM gateway behind an OpenAI-compatible URL
- Define credential lists and reference them across model entries
- Cool down failing deployments automatically
- Override per-token pricing for a custom deployment
- Run the proxy in Docker or Kubernetes with Helm

### User Intent Phrases

- How do I call Anthropic and OpenAI with the same code?
- I want one Python function that works across every LLM provider.
- How do I add automatic failover when OpenAI is down?
- How do I give each team in my company a separate LLM budget?
- How do I issue API keys with spend caps?
- I need to centralize LLM credentials so apps never see raw provider keys.
- How do I track LLM costs per user?
- Can I route the cheapest available model automatically?
- How do I add Langfuse tracing to every LLM call without changing my agent code?
- How do I run an OpenAI-compatible gateway in front of 10 different providers?
- How do I load-balance across multiple Azure OpenAI deployments?
- I want exponential backoff and retries that work the same way across providers.

### Problem Statements

- Each LLM provider has a different SDK and error model and my code is full of branches.
- A single provider outage takes down my product.
- Engineers are leaking provider API keys into repos because there is no central gateway.
- I cannot tell which team is spending how much on LLM tokens.
- I have multiple Azure deployments and no way to load-balance them.
- I am paying for premium models when a cheaper one would have answered.
- Observability tools require per-provider integration work I keep redoing.

### When to Pick This

- Pick this when you want an open-source, developer-driven gateway with a Python SDK you can embed (vs Portkey's managed-first model).
- Pick this when you need cost-based, latency-based, and usage-based routing strategies out of the box.
- Pick this when you want virtual keys with PostgreSQL-backed spend tracking you self-host.
- Pick this when most of your stack is Python and a single in-process function is preferable to a network hop.
- Pick this when you need text generation, embeddings, and image generation, but not music or video (vs ccapi).
- Pick this when you want declarative observability callbacks instead of building each integration yourself.
- Pick this when 100+ providers and broad community coverage matter more than enterprise routing primitives.

### Related Terms and Aliases

- llm proxy
- llm router
- multi-provider sdk
- model abstraction layer
- openai shim
- llm cost tracking
- llm spend management
- failover gateway
- llm load balancer
- BerriAI proxy
