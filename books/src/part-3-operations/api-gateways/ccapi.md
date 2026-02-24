# ccapi

> Unified AI API gateway for 100+ models with OpenAI-compatible endpoint

| Field | Value |
|-------|-------|
| Group | API Gateways & Model Routing |
| Type | API |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://ccapi.ai/) |

## Overview

CCAPI is a multimodal AI API gateway that aggregates multiple providers under a single OpenAI-compatible endpoint. The platform routes requests to over 100 models across seven or more providers spanning four modalities: text, image, audio, and video. CCAPI's core value proposition is migration simplicity -- existing code targeting the OpenAI API can be redirected to CCAPI by changing only the base URL to `https://api.ccapi.ai/v1` and supplying a CCAPI API key. Smart routing automatically switches between providers on failure with approximately 120 milliseconds of failover latency, maintaining a reported 99.9% success rate. The service operates on a pay-per-use billing model denominated in United States Dollars (USD) with no subscriptions or credit conversion. [1]

## Core Concepts

- **Unified Endpoint**: A single OpenAI-compatible REST API base URL (`https://api.ccapi.ai/v1`) that fronts all supported providers and modalities. Developers interact with one API surface regardless of whether the underlying model is served by OpenAI, Anthropic, Google, DeepSeek, or another provider.
- **Smart Routing**: An automatic failover mechanism that detects provider downtime or errors and reroutes requests to alternative providers. The failover occurs in approximately 120 milliseconds, which is transparent to the caller. This operates across multiple retry layers to sustain the 99.9% success rate target.
- **Multimodal Support**: CCAPI supports four modalities through dedicated endpoints -- text (chat completions), image generation, audio (text-to-speech), and video generation -- all accessible under the same base URL and authentication scheme.
- **Provider-Prefixed Model Identifiers**: Models are referenced using a `provider/model` format (for example, `anthropic/claude-4.6` or `bytedance/seedance-2`), which disambiguates models across providers and allows explicit routing to a specific backend.
- **Custom Providers**: Users can configure additional OpenAI-compatible providers with their own API keys and endpoints, extending CCAPI beyond its built-in provider catalog.
- **Pay-Per-Use Billing**: No subscriptions, no credit conversion gimmicks. A $100 deposit equals $100 of usable balance, with real-time cost tracking via the usage dashboard.

## Installation and Setup

CCAPI is a hosted API service with no local installation required. Integration uses existing OpenAI-compatible SDKs.

### Authentication

CCAPI uses bearer token authentication. API keys follow the `sk-ccapi-...` prefix convention and are obtained from the CCAPI dashboard.

```bash
export CCAPI_API_KEY="sk-ccapi-your-key-here"
```

All requests must include the `Authorization: Bearer <key>` header.

### Python (OpenAI SDK)

```bash
pip install openai
```

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.ccapi.ai/v1",
    api_key="sk-ccapi-..."
)
```

### Node.js (OpenAI SDK)

```bash
npm install openai
```

```javascript
import OpenAI from "openai";

const client = new OpenAI({
    baseURL: "https://api.ccapi.ai/v1",
    apiKey: "sk-ccapi-...",
});
```

### cURL

```bash
curl -X POST "https://api.ccapi.ai/v1/chat/completions" \
  -H "Authorization: Bearer $CCAPI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "anthropic/claude-4.6",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

Any HTTP client or SDK that speaks REST and supports the OpenAI chat completions format works with CCAPI by pointing to the `https://api.ccapi.ai/v1` base URL. [1]

## Architecture

CCAPI's architecture consists of three logical layers:

1. **API Gateway Layer**: An OpenAI-compatible REST API that accepts requests at `https://api.ccapi.ai/v1`. The gateway handles authentication (bearer tokens), request validation, rate management, and response formatting. All four modality endpoints (chat completions, image generation, audio, video generation) share the same gateway infrastructure and authentication scheme.
2. **Smart Routing Layer**: A routing and failover engine that sits between the gateway and upstream providers. When a request targets a specific provider-model pair, the routing layer forwards it to that provider. If the provider returns an error or is unreachable, the routing layer automatically retries with an alternative provider capable of serving an equivalent model, with failover latency of approximately 120 milliseconds. Multiple retry layers ensure high availability.
3. **Provider Integration Layer**: Connections to upstream AI providers (OpenAI, Anthropic, Google, DeepSeek, ByteDance, Kuaishou, Zhipu AI, MiniMax, Moonshot) plus user-configured custom providers. Each integration translates between the unified CCAPI request format and the provider's native API, handling authentication, request mapping, and response normalization.

The system is monitored 24/7 with real-time tracking of latency, success rates, and provider health across all four modalities.

## Key Features and Functionality

- **Chat Completions**: Standard text generation via `/v1/chat/completions` supporting streaming (Server-Sent Events (SSE)), function/tool calling, JSON response mode, temperature and top-p sampling, stop sequences, and maximum token limits. [2]
- **Extended Thinking**: A `thinking` parameter (`{"type": "enabled"}`) activates step-by-step reasoning mode on supported models, returning intermediate reasoning content alongside the final response. [2]
- **Vision and Image Input**: Multimodal models accept image inputs within the messages array for Optical Character Recognition (OCR), image analysis, and visual question answering.
- **Image Generation**: Endpoint at `/v1/images/generations` for generating images from text prompts through supported providers.
- **Audio (Text-to-Speech)**: Endpoint at `/v1/audio/speech` for converting text to audio using available speech synthesis models.
- **Video Generation**: Endpoint at `/v1/video/generations` for generating video content. Supported models include Seedance 2.0 (ByteDance) and Kling 3.0 (Kuaishou).
- **Prompt Caching**: Repeated prompt prefixes (such as system prompts reused across conversations) receive reduced pricing, lowering cost for high-volume applications with shared context.
- **Function/Tool Calling**: Tool definitions can be passed via the `tools` parameter with `tool_choice` controlling invocation behavior (`none`, `auto`, `required`).
- **Usage Dashboard**: Real-time monitoring of costs, latency metrics, success rates, and per-request breakdowns. Tracks spending per model and per provider.
- **Custom Provider Configuration**: Users can add any OpenAI-compatible provider with their own API keys, extending the gateway beyond the built-in provider catalog.

## Use Cases

- **Provider Migration**: Teams switching from one LLM provider to another can reroute by changing only the model identifier, with no SDK or integration code changes required. The OpenAI-compatible interface means the calling code remains identical.
- **High-Availability AI Applications**: Production systems that cannot tolerate provider outages benefit from smart routing, which automatically fails over to alternative providers within 120 milliseconds.
- **Multimodal Pipelines**: Applications that need text, image, audio, and video generation from a single integration point rather than maintaining separate SDKs and authentication for each provider.
- **Cost Optimization**: Pay-per-use pricing with transparent USD billing and the ability to route to cost-effective providers (for example, DeepSeek V4 at $0.27 per million tokens) for workloads where the lowest-cost model is sufficient.
- **Video Generation Access**: Teams needing access to Chinese AI video models (Seedance 2.0, Kling 3.0) through a familiar OpenAI-compatible interface without managing direct integrations with ByteDance or Kuaishou APIs.
- **Prototyping and Evaluation**: Rapidly testing different models from different providers (GPT-5.2, Claude 4.6, DeepSeek V4, GLM-5) against the same prompts by changing only the model parameter, enabling quick comparison without provider-specific setup.

## API Reference Summary

### Chat Completions

**Endpoint**: `POST /v1/chat/completions`

**Parameters**:
- `model` (required): Model identifier in `provider/model` format (e.g., `anthropic/claude-4.6`).
- `messages` (required): Array of message objects with `role` (`system`, `user`, `assistant`, `tool`) and `content` fields.
- `stream`: Boolean to enable SSE streaming of partial responses.
- `temperature`: Sampling temperature, range `0.0` to `2.0`, default `1.0`.
- `top_p`: Nucleus sampling threshold, range `0.0` to `1.0`, default `1.0`.
- `max_tokens`: Maximum tokens in the generated response.
- `stop`: String or array of stop sequences.
- `tools`: Array of function/tool definitions for tool calling.
- `tool_choice`: Control tool invocation (`none`, `auto`, `required`).
- `response_format`: `{"type": "json_object"}` to enforce JSON output.
- `thinking`: `{"type": "enabled"}` for extended reasoning mode.

**Response**: Chat completion object containing generated content, token usage statistics, and optional tool calls or reasoning content.

**Error Codes**:
- `400`: Bad request (malformed parameters).
- `402`: Insufficient account balance.
- `500`: Server error.

### Image Generation

**Endpoint**: `POST /v1/images/generations`

### Audio (Text-to-Speech)

**Endpoint**: `POST /v1/audio/speech`

### Video Generation

**Endpoint**: `POST /v1/video/generations`

### Available Models (Selected)

**Text Generation**:
- `openai/gpt-5.2` -- OpenAI GPT-5.2 ($2.50/$10.00 per 1M input/output tokens)
- `anthropic/claude-opus-4-6` -- Anthropic Claude Opus 4.6, 200K context ($2.50/$12.50 per 1M tokens)
- `anthropic/claude-sonnet-4-6` -- Anthropic Claude Sonnet 4.6, 200K context ($1.50/$7.50 per 1M tokens)
- `anthropic/claude-haiku-4-5` -- Anthropic Claude Haiku 4.5, 200K context ($0.50/$2.50 per 1M tokens)
- `deepseek/deepseek-v4` -- DeepSeek V4 ($0.27/1M tokens)
- `zhipu/glm-5` -- Zhipu AI GLM-5 ($0.40/1M tokens)

**Video Generation**:
- `bytedance/seedance-2` -- ByteDance Seedance 2.0 ($0.34/second)
- `kuaishou/kling-3.0` -- Kuaishou Kling 3.0 ($0.39/video)

CCAPI advertises up to 50% savings compared to direct provider pricing for certain models (notably Anthropic Claude models). [1] [2]

## Configuration and Customization

### Base URL Configuration

The single required configuration change for any OpenAI SDK-based application:

```python
# Python
client = OpenAI(
    base_url="https://api.ccapi.ai/v1",
    api_key="sk-ccapi-..."
)
```

```javascript
// Node.js
const client = new OpenAI({
    baseURL: "https://api.ccapi.ai/v1",
    apiKey: "sk-ccapi-...",
});
```

### Model Selection

Models are selected via the `model` parameter using provider-prefixed identifiers:

```python
# Route to Anthropic Claude
response = client.chat.completions.create(
    model="anthropic/claude-4.6",
    messages=[{"role": "user", "content": "Hello"}]
)

# Route to DeepSeek
response = client.chat.completions.create(
    model="deepseek/deepseek-v4",
    messages=[{"role": "user", "content": "Hello"}]
)
```

### Custom Providers

Users can configure additional OpenAI-compatible providers through the CCAPI dashboard, supplying their own API keys and endpoint URLs. This allows routing through CCAPI's unified interface to providers not in the built-in catalog.

### Usage Dashboard

The dashboard provides real-time visibility into:
- Per-request cost breakdown
- Latency metrics per model and provider
- Success rate monitoring
- API key management
- Account balance tracking

## Integration Patterns

### Drop-In OpenAI Replacement

The most common integration pattern requires changing only two values in existing OpenAI SDK code:

```python
from openai import OpenAI

# Before (direct OpenAI)
# client = OpenAI(api_key="sk-openai-...")

# After (via CCAPI)
client = OpenAI(
    base_url="https://api.ccapi.ai/v1",
    api_key="sk-ccapi-..."
)

# All existing code works unchanged
response = client.chat.completions.create(
    model="openai/gpt-5.2",
    messages=[{"role": "user", "content": "Explain quantum computing."}]
)
```

### Multi-Provider Switching

A single client instance can target different providers by changing only the model parameter:

```python
models = [
    "openai/gpt-5.2",
    "anthropic/claude-4.6",
    "deepseek/deepseek-v4",
]
for model in models:
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": "Summarize this document."}]
    )
```

### Framework Integration

Any framework built on the OpenAI SDK (LangChain, LlamaIndex, CrewAI, and others) can route through CCAPI by configuring the base URL at the client level. No framework-specific adapters are needed.

### REST/HTTP Client Integration

Any language or framework with HTTP client capabilities can call CCAPI directly:

```bash
curl -X POST "https://api.ccapi.ai/v1/chat/completions" \
  -H "Authorization: Bearer sk-ccapi-..." \
  -H "Content-Type: application/json" \
  -d '{
    "model": "anthropic/claude-sonnet-4-6",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": false
  }'
```

## Examples

### Basic Chat Completion (Python)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.ccapi.ai/v1",
    api_key="sk-ccapi-..."
)

response = client.chat.completions.create(
    model="anthropic/claude-4.6",
    messages=[{"role": "user", "content": "What is machine learning?"}]
)
print(response.choices[0].message.content)
```

### Streaming Response (Node.js)

```javascript
import OpenAI from "openai";

const client = new OpenAI({
    baseURL: "https://api.ccapi.ai/v1",
    apiKey: "sk-ccapi-...",
});

const stream = await client.chat.completions.create({
    model: "anthropic/claude-4.6",
    messages: [
        { role: "system", content: "You are a helpful assistant." },
        { role: "user", content: "Explain recursion step by step." },
    ],
    stream: true,
});

for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content;
    if (content) process.stdout.write(content);
}
```

### Tool Calling (Python)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.ccapi.ai/v1",
    api_key="sk-ccapi-..."
)

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {"type": "string", "description": "City name"}
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="openai/gpt-5.2",
    messages=[{"role": "user", "content": "What is the weather in Tokyo?"}],
    tools=tools,
    tool_choice="auto"
)
```

### Extended Thinking (cURL)

```bash
curl -X POST "https://api.ccapi.ai/v1/chat/completions" \
  -H "Authorization: Bearer sk-ccapi-..." \
  -H "Content-Type: application/json" \
  -d '{
    "model": "anthropic/claude-4.6",
    "messages": [{"role": "user", "content": "Solve this step by step: 23 * 47 + 19"}],
    "thinking": {"type": "enabled"}
  }'
```

### Video Generation (cURL)

```bash
curl -X POST "https://api.ccapi.ai/v1/video/generations" \
  -H "Authorization: Bearer sk-ccapi-..." \
  -H "Content-Type: application/json" \
  -d '{
    "model": "bytedance/seedance-2",
    "prompt": "A serene mountain lake at sunrise with mist rolling over the water"
  }'
```

### JSON Mode Response (Python)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.ccapi.ai/v1",
    api_key="sk-ccapi-..."
)

response = client.chat.completions.create(
    model="deepseek/deepseek-v4",
    messages=[{"role": "user", "content": "List three programming languages with their use cases."}],
    response_format={"type": "json_object"}
)
```

## Limitations and Considerations

- **Closed-Source Service**: CCAPI is a proprietary hosted service with no self-hosted or on-premises deployment option. All requests transit through CCAPI's infrastructure, which introduces a dependency on their availability and data handling practices.
- **Provider-Dependent Features**: Feature support (tool calling, vision, extended thinking, structured output) varies by upstream model and provider. Not all features are available across all models.
- **No Model Fine-Tuning**: CCAPI is an inference routing layer, not a model training or fine-tuning platform. Fine-tuned models must be hosted by the upstream provider and accessed through CCAPI if that provider is supported.
- **Latency Overhead**: Routing through an intermediary gateway adds network hops compared to calling a provider directly. While CCAPI reports sub-2-second latency, the additional hop may be noticeable for latency-sensitive applications where every millisecond matters.
- **Rate Limits**: Rate limits and quotas are not publicly documented in detail. Throughput may be constrained by both CCAPI's gateway limits and the underlying provider's rate limits.
- **Model Availability Lag**: New models released by providers may not be immediately available through CCAPI. There is an inherent delay between a provider launching a model and CCAPI integrating it.
- **Limited Documentation**: As a newer service, CCAPI's public documentation is less extensive than established gateways. Detailed API reference for image, audio, and video endpoints is sparse compared to the chat completions documentation.
- **Vendor Lock-In Risk**: While CCAPI uses an OpenAI-compatible interface (reducing switching cost), reliance on provider-prefixed model identifiers (`anthropic/claude-4.6`) creates a CCAPI-specific naming convention that requires mapping if migrating to another gateway.
- **Data Privacy**: All requests and responses pass through CCAPI's servers. Organizations with strict data residency or privacy requirements should evaluate CCAPI's data handling policies before routing sensitive workloads through the gateway.

## Changelog Highlights

- Launch of unified multimodal API gateway supporting text, image, audio, and video through a single endpoint.
- Smart routing with automatic provider failover in approximately 120 milliseconds.
- Integration of video generation models Seedance 2.0 (ByteDance) and Kling 3.0 (Kuaishou).
- Support for extended thinking mode on compatible models.
- Function/tool calling support across providers.
- Prompt caching with reduced pricing for repeated prefixes.
- Custom provider configuration allowing users to bring their own API keys.
- Real-time usage dashboard with cost, latency, and success rate monitoring.
- Claude model pricing at up to 50% below direct Anthropic pricing.

## Citations

- [1] CCAPI Official Website - https://ccapi.ai/
- [2] CCAPI API Documentation - https://docs.ccapi.ai/
