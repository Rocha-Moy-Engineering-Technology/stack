# ccapi

> Unified AI API gateway for 100+ models with OpenAI-compatible endpoint

| Field | Value |
|-------|-------|
| Name | ccapi |
| Group | Model Gateways |
| Type | API |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Docs | [ccapi.ai](https://ccapi.ai/) |

## Overview

CCAPI is a multimodal AI API gateway that aggregates multiple providers under a single OpenAI-compatible endpoint. The platform routes requests to over 100 models across eight or more providers spanning five modalities: text, image, audio, music, and video. CCAPI's core value proposition is migration simplicity -- existing code targeting the OpenAI API can be redirected to CCAPI by changing only the base URL to `https://api.ccapi.ai/v1` and supplying a CCAPI API key. Smart routing automatically switches between providers on failure with approximately 120 milliseconds of failover latency, maintaining a reported 99.9% success rate. The service operates on a tiered subscription model with pay-per-use billing denominated in United States Dollars (USD), offering a free tier with a $0.50 signup bonus, paid Standard and Pro plans, and custom enterprise agreements. As of March 2026, the pricing catalog covers 48 model families across text (30), image (8), audio (2), and video (8). [1]

## Core Concepts

- **Unified Endpoint**: A single OpenAI-compatible REST API base URL (`https://api.ccapi.ai/v1`) that fronts all supported providers and modalities. Developers interact with one API surface regardless of whether the underlying model is served by OpenAI, Anthropic, Google, DeepSeek, or another provider.
- **Smart Routing**: An automatic failover mechanism that detects provider downtime or errors and reroutes requests to alternative providers. The failover occurs in approximately 120 milliseconds, which is transparent to the caller. Multiple retry layers sustain the 99.9% success rate target. Multi-channel failover routes requests across multiple upstream channels per provider for higher availability. [1]
- **Multimodal Support**: CCAPI supports five modalities through dedicated endpoints -- text (chat completions), image generation, audio (text-to-speech), music generation (via Suno), and video generation -- all accessible under the same base URL and authentication scheme. [2]
- **Provider-Prefixed Model Identifiers**: Models are referenced using a `provider/model` format (for example, `anthropic/claude-sonnet-4-6` or `bytedance/seedance-2`), which disambiguates models across providers and allows explicit routing to a specific backend.
- **Custom Providers**: Users can configure additional OpenAI-compatible providers with their own API keys and endpoints, extending CCAPI beyond its built-in provider catalog.
- **Subscription Tiers**: Four plans -- Free, Standard ($15.83/month billed annually), Pro ($49.17/month billed annually), and Custom (negotiated) -- each unlocking progressively deeper model discounts (up to 40%, 50%, 60%, and negotiated respectively), higher rate limits, and additional operational features. [3]
- **Pay-Per-Use Billing**: No credit conversion. A $100 deposit equals $100 of usable balance, with real-time cost tracking via the usage dashboard.
- **OpenClaw Integration**: CCAPI provides a dedicated integration path for OpenClaw (open-source AI agent framework with 191K+ GitHub stars), enabling agents to route all Large Language Model (LLM) calls through CCAPI by changing environment variables -- no code changes required. [4]

## Architecture

CCAPI's architecture consists of three logical layers:

1. **API Gateway Layer**: An OpenAI-compatible REST API that accepts requests at `https://api.ccapi.ai/v1`. The gateway handles authentication (bearer tokens), request validation, rate management based on subscription tier, and response formatting. All modality endpoints (chat completions, image generation, audio, music, video generation) share the same gateway infrastructure and authentication scheme.
2. **Smart Routing Layer**: A routing and failover engine that sits between the gateway and upstream providers. When a request targets a specific provider-model pair, the routing layer forwards it to that provider. If the provider returns an error or is unreachable, the routing layer automatically retries with an alternative provider capable of serving an equivalent model, with failover latency of approximately 120 milliseconds. Multi-channel failover distributes requests across multiple upstream channels per provider for higher availability.
3. **Provider Integration Layer**: Connections to upstream AI providers (OpenAI, Anthropic, Google, DeepSeek, ByteDance, Kuaishou, Zhipu AI, MiniMax, Moonshot, Qwen, Suno, xAI/Grok, Midjourney) plus user-configured custom providers. Each integration translates between the unified CCAPI request format and the provider's native API, handling authentication, request mapping, and response normalization.

The system is monitored 24/7 with real-time tracking of latency, success rates, and provider health across all modalities.

## Key Features and Functionality

- **Chat Completions**: Standard text generation via `/v1/chat/completions` supporting streaming (Server-Sent Events (SSE)), function/tool calling, JavaScript Object Notation (JSON) response mode, temperature and top-p sampling, stop sequences, and maximum token limits. [2]
- **Extended Thinking**: A `thinking` parameter (`{"type": "enabled"}`) activates step-by-step reasoning mode on supported models, returning intermediate reasoning content alongside the final response. [2]
- **Vision and Image Input**: Multimodal models accept image inputs within the messages array for Optical Character Recognition (OCR), image analysis, and visual question answering.
- **Image Generation**: Endpoint at `/v1/images/generations` for generating images from text prompts. Supported providers include Midjourney, Google Gemini Image, and ByteDance Seedream. [2]
- **Audio (Text-to-Speech)**: Endpoint at `/v1/audio/speech` for converting text to audio using available speech synthesis models.
- **Music Generation**: Integration with Suno for AI music generation, including the Chirp model family, with operation-based pricing. [2] [3]
- **Video Generation**: Endpoint at `/v1/video/generations` for generating video content. Supported models include Seedance 2.0 (ByteDance), Kling 3.0 Standard/Omni/Action Control (Kuaishou), Veo 3.1 Stream (Google), Sora 2/Sora 2 Pro/Sora 2 Lite (OpenAI), Midjourney Video, and Grok Video (xAI). Sora 2 supports flexible 4-second, 8-second, and 12-second runtimes. Kling 3.0 supports 4K resolution at 60 frames per second (fps) with multi-shot capability. Seedance 2.0 supports 2K resolution at 24 fps with audio sync. [2] [5]
- **Prompt Caching**: Repeated prompt prefixes (such as system prompts reused across conversations) receive reduced pricing, lowering cost for high-volume applications with shared context. [3]
- **Function/Tool Calling**: Tool definitions can be passed via the `tools` parameter with `tool_choice` controlling invocation behavior (`none`, `auto`, `required`).
- **File-to-URL API**: Upload files and receive temporary Uniform Resource Locators (URLs) for multimodal AI models, with per-tier upload quotas, download limits, and 24-hour automatic file expiry. Available on Standard tier and above. [5]
- **Webhook Notifications**: Asynchronous event notifications for completed operations, available on Standard tier and above. [3]
- **Usage Dashboard**: Real-time monitoring of costs, latency metrics, success rates, and per-request breakdowns. Tracks spending per model and per provider. [1]
- **Custom Provider Configuration**: Users can add any OpenAI-compatible provider with their own API keys, extending the gateway beyond the built-in provider catalog.
- **Team Collaboration**: Standard tier includes 3 team seats, Pro includes 8, and Custom tier offers unlimited seats. [3]

## Use Cases

- **Provider Migration**: Teams switching from one LLM provider to another can reroute by changing only the model identifier, with no SDK or integration code changes required. The OpenAI-compatible interface means the calling code remains identical.
- **High-Availability AI Applications**: Production systems that cannot tolerate provider outages benefit from smart routing, which automatically fails over to alternative providers within 120 milliseconds.
- **Multimodal Pipelines**: Applications that need text, image, audio, music, and video generation from a single integration point rather than maintaining separate SDKs and authentication for each provider.
- **Cost Optimization**: Tiered subscription discounts (up to 60% off on the Pro plan for Anthropic Claude models) combined with pay-per-use pricing enable significant savings. For example, DeepSeek V3 at $0.27 per million input tokens can reduce costs by 97% compared to Claude Opus for equivalent workloads. [3] [4]
- **AI Agent Cost Reduction**: OpenClaw users report monthly LLM costs of $623 to $3,600 with direct provider APIs. Routing through CCAPI to budget models (DeepSeek, GLM-5, MiniMax M2.5) can reduce costs to under $20 per month at 50 million tokens per day. [4]
- **Video Generation Access**: Teams needing access to video generation models (Seedance 2.0, Kling 3.0, Veo 3.1, Sora 2, Midjourney, Grok Video) through a familiar OpenAI-compatible interface without managing direct integrations with each provider. [5]
- **Prototyping and Evaluation**: Rapidly testing different models from different providers against the same prompts by changing only the model parameter, enabling quick comparison without provider-specific setup.

## API Reference Summary

### Chat Completions

**Endpoint**: `POST /v1/chat/completions`

**Parameters**:
- `model` (required): Model identifier in `provider/model` format (e.g., `anthropic/claude-sonnet-4-6`).
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

Supported providers: Midjourney, Google Gemini Image, ByteDance Seedream. [2]

### Audio (Text-to-Speech)

**Endpoint**: `POST /v1/audio/speech`

### Music Generation

**Provider**: Suno (Chirp model family). Operation-based pricing with core music generation and advanced operations. [2] [3]

### Video Generation

**Endpoint**: `POST /v1/video/generations`

Supported models include Seedance 2.0, Kling 3.0 (Standard/Omni/Action Control), Veo 3.1 Stream, Sora 2 (Classic/Flexible/Lite/Pro/Pro HD), Midjourney Video, and Grok Video. Video billing varies by model: per-second (Seedance, Veo 3.1 Stream, Sora 2 Flexible), per-video (Kling, Sora 2 Classic), or packaged by resolution and duration (Grok Video). [2] [5]

### Available Models (Selected)

**Text Generation**:
- `openai/gpt-5.2` -- OpenAI GPT-5.2
- `openai/gpt-5.4` -- OpenAI GPT-5.4 (threshold pricing for different input context sizes)
- `anthropic/claude-opus-4-6` -- Anthropic Claude Opus 4.6, 200K context (Pro: $2.00/$10.00 per 1M tokens)
- `anthropic/claude-sonnet-4-6` -- Anthropic Claude Sonnet 4.6, 200K context (Pro: $1.20/$6.00 per 1M tokens)
- `anthropic/claude-haiku-4-5` -- Anthropic Claude Haiku 4.5, 200K context (Pro: $0.40/$2.00 per 1M tokens)
- `google/gemini-2.5-pro` -- Google Gemini 2.5 Pro (Pro: $1.25/$7.50 per 1M tokens)
- `google/gemini-3` -- Google Gemini 3
- `openai/gpt-4o` -- OpenAI GPT-4o (Pro: $1.25/$5.00 per 1M tokens)
- `deepseek/deepseek-v4` -- DeepSeek V4 ($0.27/$1.10 per 1M tokens)
- `deepseek/deepseek-v3.2` -- DeepSeek V3.2
- `zhipu/glm-5` -- Zhipu AI GLM-5 ($0.40/$1.80 per 1M tokens)
- `minimax/minimax-m2-5` -- MiniMax M2.5 ($0.21/$0.84 per 1M tokens)
- `moonshot/kimi-k2.5` -- Moonshot Kimi K2.5 ($0.55/$2.19 per 1M tokens)
- `qwen/qwen3.5-plus` -- Qwen 3.5 Plus (with prompt caching)

**Video Generation**:
- `bytedance/seedance-2` -- ByteDance Seedance 2.0 (2K, 4-15s, 24 fps, audio sync, $0.34/second)
- `kuaishou/kling-3.0` -- Kuaishou Kling 3.0 (4K, 3-15s, 60 fps, multi-shot, $0.39/video)
- `google/veo-3.1` -- Google Veo 3.1 Stream (runtime-based billing, Pro from $0.09/second)
- `openai/sora-2` -- OpenAI Sora 2 (flexible 4/8/12s runtimes, Pro from $0.06/second or per-video)
- `midjourney/video` -- Midjourney Video
- `grok/video` -- Grok Video (packaged by 480p/720p and 6/10/15s durations)

**Image Generation**:
- Midjourney, Google Gemini Image, ByteDance Seedream

**Audio/Music**:
- Suno (Chirp family, operation-based pricing, Pro: $0.07/call)
- Producer (audio generation)

CCAPI advertises up to 60% savings compared to direct provider pricing on the Pro tier, with Anthropic Claude models receiving the deepest discounts. [1] [3]

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

### Subscription Tiers

| Feature | Free | Standard | Pro | Custom |
|---------|------|----------|-----|--------|
| Price | $0 | $15.83/mo (annual) | $49.17/mo (annual) | Negotiated |
| API Discount | Up to 40% | Up to 50% | Up to 60% | Negotiated |
| Models | Standard | All + Premium | All + Premium + Beta | All + Priority Access |
| Rate Limit (RPM) | 30 | 600 | 3,000 | Custom |
| Concurrency | 2 | 10 | 30 | Unlimited |
| Team Seats | -- | 3 | 8 | Unlimited |
| File-to-URL API | -- | Yes | Yes | Yes |
| Webhooks | -- | Yes | Yes | Yes |
| Log Retention | 24 hours | 7 days | 30 days | 365 days |
| Support | Community | Email | Priority Email | Dedicated Manager |
| SLA | -- | -- | -- | 99.9%+ |
| Private Deployment | -- | -- | -- | Available |

All plans include a $0.50 signup bonus and direct USD billing. Annual billing saves two months (Standard: $190/year, Pro: $590/year). [3]

### Model Selection

Models are selected via the `model` parameter using provider-prefixed identifiers:

```python
# Route to Anthropic Claude
response = client.chat.completions.create(
    model="anthropic/claude-sonnet-4-6",
    messages=[{"role": "user", "content": "Hello"}]
)

# Route to DeepSeek for cost efficiency
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
    "anthropic/claude-sonnet-4-6",
    "deepseek/deepseek-v4",
    "zhipu/glm-5",
    "minimax/minimax-m2-5",
]
for model in models:
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": "Summarize this document."}]
    )
```

### OpenClaw Agent Integration

Connect OpenClaw to CCAPI by setting environment variables in the `.env` file:

```bash
# CCAPI Gateway Configuration
OPENCLAW_API_KEY=sk-your-ccapi-api-key
OPENCLAW_MODEL=deepseek/deepseek-chat
OPENCLAW_BASE_URL=https://api.ccapi.ai/api/v1

# Optional: fallback model
OPENCLAW_FALLBACK_MODEL=openai/gpt-4o-mini
```

No code changes are required. OpenClaw treats CCAPI as a standard OpenAI endpoint. [4]

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
    model="anthropic/claude-sonnet-4-6",
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
    model: "anthropic/claude-sonnet-4-6",
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
    "model": "anthropic/claude-sonnet-4-6",
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

- **Closed-Source Service**: CCAPI is a proprietary hosted service with no self-hosted or on-premises deployment option (except at the Custom tier with private deployment). All requests transit through CCAPI's infrastructure, which introduces a dependency on their availability and data handling practices.
- **Provider-Dependent Features**: Feature support (tool calling, vision, extended thinking, structured output) varies by upstream model and provider. Not all features are available across all models.
- **No Model Fine-Tuning**: CCAPI is an inference routing layer, not a model training or fine-tuning platform. Fine-tuned models must be hosted by the upstream provider and accessed through CCAPI if that provider is supported.
- **Latency Overhead**: Routing through an intermediary gateway adds network hops compared to calling a provider directly. While CCAPI reports sub-2-second latency, the additional hop may be noticeable for latency-sensitive applications where every millisecond matters.
- **Rate Limits**: Rate limits are tier-dependent (30 Requests Per Minute (RPM) on Free, 600 on Standard, 3,000 on Pro). Throughput may be further constrained by the underlying provider's rate limits. [3]
- **Model Availability Lag**: New models released by providers may not be immediately available through CCAPI. There is an inherent delay between a provider launching a model and CCAPI integrating it, though the changelog shows rapid integration cadence.
- **Limited Documentation Depth**: The API documentation (hosted on docs.ccapi.ai) primarily covers endpoint specifications per provider. Detailed guides for image, audio, music, and video endpoints are less extensive compared to the chat completions documentation. [2]
- **Vendor Lock-In Risk**: While CCAPI uses an OpenAI-compatible interface (reducing switching cost), reliance on provider-prefixed model identifiers (`anthropic/claude-sonnet-4-6`) creates a CCAPI-specific naming convention that requires mapping if migrating to another gateway.
- **Data Privacy**: All requests and responses pass through CCAPI's servers. Organizations with strict data residency or privacy requirements should evaluate CCAPI's data handling policies before routing sensitive workloads through the gateway.
- **Free Tier Constraints**: The free tier has significant limitations: 30 RPM, 2 concurrent requests, 24-hour log retention, community-only support, and access limited to standard models. [3]

## Changelog Highlights

- **v1.8.0** (March 9, 2026): Sora 2 flexible 4/8/12-second runtimes, Sora 2 Pro short-form resolution options, Grok Video packaged pricing by resolution and duration. [5]
- **v1.7.0** (March 8, 2026): Kling 3.0 Standard/Omni/Action Control documentation, expanded Kling video request options, GPT-5.4 with threshold pricing for input context sizes. [5]
- **v1.6.0** (March 5, 2026): Up to 50% off OpenAI and Google Gemini models, up to 60% off Anthropic Claude models with tiered discounts, multi-channel failover for OpenAI and Gemini text models, Sora 2 Lite and Sora 2 Pro HD, model ID resolution fixes. [5]
- **v1.5.0** (February 22, 2026): File-to-URL API for multimodal file uploads with per-tier quotas and 24-hour automatic expiry. [5]
- **v1.4.0** (February 20, 2026): Subscription plans (Free, Standard, Pro, Custom tiers), billing dashboard and usage tracking. [5]
- **v1.3.0** (February 19, 2026): Google Veo 3.1 video generation, GPT-5.2 Codex, Qwen3.5-Plus with prompt caching, GLM-4.7, Kimi K2.5. [5]
- **v1.2.0** (February 17, 2026): OpenAI Sora 2 video generation, redesigned models mega dropdown, mobile responsiveness improvements, auto-refund for failed video generation jobs. [5]

## Citations

- [1] CCAPI Official Website - https://ccapi.ai/
- [2] CCAPI API Documentation - https://docs.ccapi.ai/
- [3] CCAPI Pricing - https://ccapi.ai/pricing
- [4] CCAPI OpenClaw Integration - https://ccapi.ai/openclaw
- [5] CCAPI Changelog - https://ccapi.ai/changelog

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- ccapi
- ccapi.ai
- unified multimodal api
- openai-compatible gateway
- smart routing
- 120ms failover
- provider-prefixed model
- multi-channel failover
- image generation api
- music generation api
- video generation api
- audio tts
- suno
- seedance
- kling
- veo
- sora
- midjourney video
- grok video
- deepseek
- glm
- minimax
- qwen
- kimi
- openclaw
- file-to-url api
- prompt caching
- pay-per-use
- subscription tiers
- thinking parameter
- extended thinking

### Verb-Noun Tasks

- Swap an OpenAI SDK base URL to route through ccapi
- Generate a Sora 2, Kling 3.0, Veo 3.1, or Seedance video from a prompt
- Compose music with Suno via an OpenAI-compatible call
- Generate images with Midjourney or Seedream through the same endpoint
- Switch model providers by changing only the `provider/model` identifier
- Enable extended thinking with `{"thinking": {"type": "enabled"}}`
- Upload a file and receive a temporary URL for multimodal models
- Receive a webhook when a long-running operation completes
- Route OpenClaw agents through ccapi by changing `.env` variables
- Track per-request cost and balance in the usage dashboard
- Add a custom OpenAI-compatible provider to extend the catalog
- Subscribe to a tier to unlock deeper API discounts

### User Intent Phrases

- I want one API key that generates text, images, music, and video.
- How do I call Sora 2, Veo 3.1, and Kling 3.0 through one endpoint?
- How do I get Anthropic Claude with up to 60% off?
- Can I switch from OpenAI to DeepSeek without changing my code?
- How do I generate AI music with Suno via an HTTP API?
- How do I add fallback between providers without writing routing code?
- Where can I find a unified video generation API?
- How do I migrate OpenClaw to a cheaper LLM by editing only env vars?
- How do I send a file to a multimodal model that requires a URL?
- I want extended thinking on Claude through a single base URL.
- How do I track LLM spend across multiple providers in one dashboard?

### Problem Statements

- I need video generation but I do not want to integrate Sora, Kling, Veo, and Seedance separately.
- Provider outages stall my product and I have no automatic failover.
- My agent loop is too expensive on frontier closed models.
- I need a single billing relationship instead of 8 different vendor invoices.
- I want to generate music and images alongside text without 3 SDKs.
- My team needs prompt caching discounts but the underlying providers only offer them piecemeal.

### When to Pick This

- Pick this when you need non-text modalities — image, music, and video generation — through the same gateway (vs LiteLLM and Portkey, which are text/embedding focused).
- Pick this when access to Sora 2, Kling 3.0, Veo 3.1, Seedance 2.0, Midjourney Video, and Grok Video matters.
- Pick this when sub-200ms automatic provider failover is required without writing routing code.
- Pick this when Chinese/open-weight providers (DeepSeek, GLM, MiniMax, Kimi, Qwen) at deep discounts are a primary cost lever.
- Pick this when you want a managed, closed-source, pay-as-you-go service rather than a self-hosted gateway.
- Pick this when OpenClaw agents need a drop-in cheaper backend by env-var change only.
- Pick this when consolidated multimodal billing in USD with a $0.50 signup credit is preferable to per-provider accounts.

### Related Terms and Aliases

- multimodal ai gateway
- ai aggregator api
- ai provider switchboard
- video generation api
- music api
- multi-provider llm proxy
- text-image-audio-video api
- ai api marketplace
- openai-compatible aggregator
- closed-source llm gateway
