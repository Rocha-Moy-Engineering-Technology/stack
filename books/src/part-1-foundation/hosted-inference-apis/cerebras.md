# Cerebras

> AI inference API on custom wafer-scale engine hardware

| Field | Value |
|-------|-------|
| Name | Cerebras |
| Group | Hosted Inference APIs |
| Type | API/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Docs | [inference-docs.cerebras.ai](https://inference-docs.cerebras.ai/) |

## Overview

Cerebras is an AI inference platform built on custom wafer-scale engine (WSE) hardware. Instead of traditional GPU clusters, Cerebras uses its proprietary CS-3 chip architecture where an entire silicon wafer acts as a single processor, eliminating inter-chip communication bottlenecks. The platform exposes a cloud API that is OpenAI-compatible, allowing developers to migrate from OpenAI endpoints with minimal code changes. Cerebras targets use cases where inference speed is critical, with production models achieving up to approximately 3,000 tokens per second for the gpt-oss-120b model and approximately 2,200 tokens per second for llama3.1-8b.

The platform serves both general-purpose chat completions and advanced capabilities including reasoning models with configurable effort, structured JSON outputs, parallel function calling, prompt caching, batch processing, and a planning and optimization framework called CePO (Cerebras Planning & Optimization).

## Core Concepts

- **Wafer-Scale Engine (WSE):** Cerebras hardware is based on CS-3 wafer-scale chips where an entire silicon wafer acts as a single processor. This eliminates multi-chip communication overhead found in traditional GPU clusters, enabling faster sequential token generation.
- **OpenAI-Compatible API:** The chat completions endpoint follows the same request and response schema as the OpenAI API. Applications using the OpenAI Python or JavaScript SDKs can switch to Cerebras by changing the base URL to `https://api.cerebras.ai/v1` and providing a Cerebras API key.
- **Service Tiers:** Four configurable tiers (priority, default, auto, flex) control request scheduling and queue behavior. Priority offers lowest latency for dedicated endpoints; default provides standard processing; auto dynamically selects the best available tier; flex offers lowest cost with potential queuing during peak periods.
- **Reasoning Effort and Format:** For supported models, the `reasoning_effort` parameter (low, medium, high) controls chain-of-thought depth. The `reasoning_format` parameter (parsed, raw, hidden, none) controls how reasoning content appears in responses. Parsed mode separates reasoning into a dedicated field; raw prepends it to content; hidden excludes it from output while still counting tokens.
- **Prompt Caching:** Automatic server-side caching of prompt prefixes in 128-token blocks. Cached data persists for 5 minutes (guaranteed) up to 1 hour. Exact character-level prefix matching is required. Cached tokens count toward rate limits but incur no additional fees.
- **CePO (Cerebras Planning & Optimization):** A framework built on the open-source OptiLLM library that adds advanced reasoning to Llama models via test-time compute. CePO uses four stages: planning, execution (multiple responses), analysis (inconsistency detection), and best-of-N selection with confidence scoring.
- **Model Compression:** Cerebras uses selective weight-only quantization (FP16/FP8) during storage with sensitive layers at full precision. Dequantization happens on the fly, so compute operations run in high precision. Activations and key-value cache remain unquantized. No models are pruned on public endpoints.

## Architecture

Cerebras operates as a managed cloud inference service with three layers:

1. **Client Layer:** Applications interact through the Python SDK (`cerebras.cloud.sdk`), the JavaScript SDK (`@cerebras/cerebras_cloud_sdk`), or direct REST calls against `https://api.cerebras.ai/v1/`.
2. **API Gateway:** Handles authentication (Bearer token), request validation, service tier routing, queue management, and API version negotiation. The gateway exposes an OpenAI-compatible interface. Rate limiting uses a token bucketing algorithm at the organization level, tracking both requests and tokens across per-minute, per-hour, and per-day windows.
3. **Inference Engine:** Requests are dispatched to CS-3 wafer-scale engine hardware for model execution. The hardware architecture eliminates multi-chip communication overhead. Prompt caching is handled at this layer, reusing previously computed key-value states for shared prompt prefixes.

The response includes `time_info` metrics (`queue_time`, `prompt_time`, `completion_time`, `total_time`) exposing the performance characteristics of each layer.

## Key Features

- **High-Speed Inference:** Wafer-scale hardware delivers fast token generation. Production speeds: gpt-oss-120b at approximately 3,000 tokens/s, llama3.1-8b at approximately 2,200 tokens/s, Qwen 3 235B at approximately 1,400 tokens/s, Z.ai GLM 4.7 at approximately 1,000 tokens/s.
- **Prompt Caching:** Automatic prefix caching in 128-token blocks with 5-minute to 1-hour retention. Tracked via `cached_tokens` in `usage.prompt_tokens_details`. Supported on gpt-oss-120b, zai-glm-4.7, and qwen-3-235b-a22b-instruct-2507.
- **Structured Outputs:** The `response_format` parameter supports `text`, `json_object`, and `json_schema` modes with constrained decoding when `strict: true`. JSON schema enforcement ensures model output conforms to user-defined schemas.
- **Streaming:** Real-time token streaming via Server-Sent Events (SSE). Multi-token streaming delivers batches of 200 events per second. Supported for all models including structured outputs.
- **Tool Use and Function Calling:** The `tools` parameter accepts function definitions. Parallel tool calls are supported via `parallel_tool_calls` (default: true). The model can request multiple function executions in a single response turn. Constrained decoding ensures valid tool call JSON.
- **Reasoning Models:** GPT-OSS-120b supports `reasoning_effort` (low/medium/high). Z.ai GLM 4.7 supports `disable_reasoning` (boolean). Both support `reasoning_format` (parsed/raw/hidden/none) controlling reasoning visibility. Logprobs are separated into `reasoning_logprobs` when using parsed format.
- **Predicted Outputs:** The `prediction` parameter supplies an expected output, enabling the engine to accelerate generation when the prediction is close to the actual response.
- **Batch API:** Asynchronous processing of up to 50,000 requests per batch with 50% cost savings. Uses a Files API for input/output management.
- **Metrics API:** Prometheus-compatible monitoring for dedicated endpoints. Tracks request counts, token throughput, latency percentiles (p50/p90/p95/p99), queue time, cache hit rates, and endpoint health. Scrape interval: 60 seconds.
- **CePO Framework:** Test-time compute framework for Llama models using planning, multi-execution, inconsistency analysis, and best-of-N confidence scoring. Built on OptiLLM.
- **Completions Endpoint:** Separate `POST /v1/completions` endpoint for single-turn text generation with prompt-based input (strings, token arrays). Supports echo, grammar roots, and raw token return.

## Use Cases

- **Low-Latency Chat Applications:** Wafer-scale inference speed suits real-time conversational interfaces where response latency directly affects user experience.
- **Batch Inference Pipelines:** The Batch API and flex service tier enable cost-effective processing (50% savings) of large volumes of requests sharing common prefixes.
- **Structured Data Extraction:** JSON schema-enforced outputs with constrained decoding are useful for entity extraction, form parsing, and data normalization.
- **Agentic Tool Use:** Parallel function calling with constrained decoding supports agentic workflows where the model orchestrates multiple external tool invocations per turn.
- **Reasoning-Heavy Tasks:** Configurable reasoning effort on gpt-oss-120b allows tuning the cost-accuracy tradeoff for math, coding, and multi-step analysis tasks.
- **Enhanced Reasoning with CePO:** The CePO framework adds planning and optimization capabilities to Llama models for tasks requiring iterative reasoning and self-correction.
- **Code Generation:** Integrations with coding tools (Aider, Cline, OpenCode, RooCode, VS Code, KiloCode) enable AI pair programming with Cerebras inference speed.

## API Reference

### Endpoints

```
POST https://api.cerebras.ai/v1/chat/completions
POST https://api.cerebras.ai/v1/completions
```

### Authentication

```
Authorization: Bearer <CEREBRAS_API_KEY>
```

### Available Models

**Production Models:**

- `llama3.1-8b` -- 8 billion parameters, approximately 2,200 tokens/s, FP16 precision
- `gpt-oss-120b` -- 120 billion parameters, approximately 3,000 tokens/s, FP16/FP8 (weight-only quantization)

**Preview Models:**

- `qwen-3-235b-a22b-instruct-2507` -- 235 billion parameters, approximately 1,400 tokens/s, FP16/FP8
- `zai-glm-4.7` -- 355 billion parameters, approximately 1,000 tokens/s, FP16/FP8

### Chat Completions Request Parameters

**Required:**

- `model` (string): Model identifier.
- `messages` (array): Message objects with `role` (system, user, assistant) and `content`.

**Response Control:**

- `max_completion_tokens` (integer): Maximum tokens to generate.
- `temperature` (float, 0-1.5): Sampling temperature.
- `top_p` (float, 0-1): Nucleus sampling threshold.
- `stream` (boolean): Enable streaming responses.
- `stop` (string or array): Up to 4 stop sequences.
- `seed` (integer): Deterministic sampling seed.

**Reasoning:**

- `reasoning_effort` (string: low, medium, high): Chain-of-thought depth (gpt-oss-120b only).
- `reasoning_format` (string: parsed, raw, hidden, none): Controls reasoning visibility in response.
- `clear_thinking` (boolean): Include thinking from previous turns (zai-glm-4.7 only).
- `disable_reasoning` (boolean): Disable reasoning (zai-glm-4.7 only).

**Structured Output:**

- `response_format` (object): One of `text`, `json_object`, or `json_schema` with schema definition.

**Tool Use:**

- `tools` (array): Function definitions for tool calling.
- `tool_choice` (string or object): Control tool selection (none, auto, required, or specific tool).
- `parallel_tool_calls` (boolean, default: true): Allow multiple tool calls per response.

**Advanced:**

- `logprobs` (boolean): Return log probabilities.
- `top_logprobs` (integer, 0-20): Number of top log probabilities per token.
- `prediction` (object): Predicted output for accelerated generation.
- `user` (string): End-user identifier.

**Service:**

- `service_tier` (string: priority, default, auto, flex): Request scheduling tier.
- `queue_threshold` (integer, 50-20000 ms): Maximum acceptable queue time before rejection. Applies to flex and auto tiers.

### Response Structure

- `id` (string): Unique completion identifier.
- `choices` (array): Completion choices with `message` (role, content), `finish_reason` (stop, length, content_filter, tool_calls), and optional `tool_calls`.
- `usage` (object): `prompt_tokens`, `completion_tokens`, `total_tokens`, and `prompt_tokens_details.cached_tokens`.
- `time_info` (object): `queue_time`, `prompt_time`, `completion_time`, `total_time`.
- `service_tier_used` (string): Actual tier used when auto was selected.

### Unsupported OpenAI Parameters

These parameters return 400 errors: `frequency_penalty`, `logit_bias`, `presence_penalty`.

### Error Codes

- 400 BadRequestError: Malformed request parameters.
- 401 AuthenticationError: Invalid or missing API credentials.
- 402 PaymentRequired: Billing issue.
- 403 PermissionDeniedError: Insufficient access rights.
- 404 NotFoundError: Resource not found.
- 422 UnprocessableEntityError: Request cannot be processed.
- 429 RateLimitError: Rate limit exceeded; back off and retry.
- 500 InternalServerError: Server-side failure.
- 503 ServiceUnavailable: Temporarily unavailable.

The SDK automatically retries connection errors, 408, 429, and 5xx responses up to 2 times. Default request timeout is 1 minute.

## Configuration

### Environment Variables

- `CEREBRAS_API_KEY` -- API key for authentication (required).

### Service Tier Selection

- **priority:** Highest priority, requests processed first. Dedicated endpoints only (private preview).
- **default:** Standard priority processing. Applied automatically when no tier is specified.
- **auto:** Dynamically selects highest available tier. Response includes `service_tier_used` field.
- **flex:** Lowest priority, requests processed last. Independent higher rate limits. Suitable for batch workloads.

All tiers bill identically during preview. The `queue_threshold` header (50-20000 ms) applies to flex and auto tiers, rejecting requests exceeding the wait threshold.

### Rate Limits

Rate limits apply at the organization level using token bucketing.

**Free Tier:**

- gpt-oss-120b: 64K TPM, 1M TPH/TPD, 30 RPM
- llama3.1-8b: 60K TPM, 1M TPH/TPD, 30 RPM
- qwen-3-235b-a22b-instruct-2507: 60K TPM, 1M TPH/TPD, 30 RPM
- zai-glm-4.7: 60K TPM, 1M TPH/TPD, 10 RPM

**PayGo Tier:**

- gpt-oss-120b: 1M TPM, 1K RPM
- llama3.1-8b: 2M TPM, 2K RPM
- qwen-3-235b-a22b-instruct-2507: 250K TPM, 250 RPM
- zai-glm-4.7: 500K TPM, 500 RPM

Rate limit headers include `x-ratelimit-remaining-tokens-minute` and reset timing information. Exceeding limits returns HTTP 429.

### API Versioning

API Version 2 is available for testing via header (introduced 2026-01-21). It introduces stricter validation for structured outputs, tool calling, reasoning models, and Unicode token handling. Becomes the default on July 21, 2026.

## Integration Patterns

### Drop-In Replacement for OpenAI

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.cerebras.ai/v1",
    api_key="your_cerebras_api_key"
)

response = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[{"role": "user", "content": "Explain wafer-scale computing."}]
)
```

For gpt-oss-120b, system messages act at the developer level with stronger influence than in the OpenAI API.

### Framework Integrations

Cerebras supports 50+ integrations across categories:

- **Agentic Frameworks:** AG2, Agno, Browser-Use, CrewAI, Stagehand
- **AI Development Kits:** Vercel AI SDK, AI Suite, Milvus
- **Coding Tools:** Aider, Cline, OpenCode, RooCode, VS Code, KiloCode
- **LLM Application Frameworks:** Instructor, LangChain, LangGraph, Pydantic AI, Llama Stack
- **Observability:** Braintrust, Langfuse, Opik, Weave, Cloudflare AI Gateway, Kong API Gateway
- **Real-Time Audio:** Cartesia, LiveKit, ElevenLabs
- **Multi-LLM Management:** LiteLLM, OpenRouter, AWS Marketplace
- **No-Code Platforms:** Dify, Flowise, FlutterFlow
- **Chatbot Platforms:** Poe
- **Containerization:** Docker

Any framework supporting OpenAI-compatible endpoints can be configured to use the Cerebras API by setting the base URL.

## Examples

### Basic Chat Completion (Python SDK)

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

chat_completion = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[{"role": "user", "content": "Hello!"}]
)

print(chat_completion.choices[0].message.content)
```

### Streaming Response

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

stream = client.chat.completions.create(
    model="llama3.1-8b",
    messages=[{"role": "user", "content": "Write a short poem."}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Structured Output with JSON Schema

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

response = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[
        {"role": "user", "content": "Extract the name and age from: John is 30 years old."}
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "person_info",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "age": {"type": "integer"}
                },
                "required": ["name", "age"]
            }
        }
    }
)

print(response.choices[0].message.content)
```

### Function Calling

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name"
                    }
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[{"role": "user", "content": "What is the weather in San Francisco?"}],
    tools=tools,
    tool_choice="auto"
)

print(response.choices[0].message.tool_calls)
```

### Reasoning with Configurable Effort and Format

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

response = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[
        {"role": "user", "content": "Prove that the square root of 2 is irrational."}
    ],
    reasoning_effort="high",
    reasoning_format="parsed"
)

# Access reasoning separately from content
print("Reasoning:", response.choices[0].message.reasoning)
print("Answer:", response.choices[0].message.content)
```

### CePO with OptiLLM

```bash
# Start OptiLLM proxy with CePO approach
optillm --base-url https://api.cerebras.ai --approach cepo

# Optional: enable intermediate state logging
optillm --base-url https://api.cerebras.ai --approach cepo --cepo_print_output true
```

Then connect to the OptiLLM proxy (default localhost:8000) using standard OpenAI-compatible requests with the `llama3.1-8b` model.

## Limitations

- **Closed Source Hardware and Software:** The wafer-scale engine and inference runtime are proprietary. Users cannot self-host or inspect the inference pipeline.
- **Limited Model Catalog:** Only four models are available at any given time (two production, two preview). The catalog is smaller than general-purpose GPU cloud platforms.
- **No Fine-Tuning:** The platform provides inference only. Users cannot fine-tune or train custom models through the cloud API.
- **Preview Model Stability:** Models marked as preview (qwen-3-235b-a22b-instruct-2507, zai-glm-4.7) may be discontinued with short notice and should not be used in production.
- **Model-Specific Parameters:** Some parameters are model-specific (`reasoning_effort` for gpt-oss-120b only; `clear_thinking` and `disable_reasoning` for zai-glm-4.7 only), requiring conditional logic when supporting multiple models.
- **Unsupported OpenAI Parameters:** `frequency_penalty`, `logit_bias`, and `presence_penalty` return 400 errors, limiting some sampling strategies available on OpenAI.
- **System Message Behavior:** For gpt-oss-120b, system messages have stronger influence than in the OpenAI API (developer-level), which may produce different outputs with identical prompts.
- **Prompt Caching Constraints:** Requires exact character-level prefix matching. Even minor variations (timestamps, dynamic content at the start) prevent cache hits.
- **Regional Availability:** As a specialized hardware platform, availability may be constrained by data center locations and capacity.

## Changelog

- **2026-01-22:** Metrics API launched with Prometheus-compatible monitoring for dedicated endpoints.
- **2026-01-21:** API Version 2 available for testing. Stricter validation for structured outputs, tool calling, reasoning, and Unicode. Default July 21, 2026.
- **2026-01-14:** Service Tiers feature launched (priority, default, auto, flex).
- **2026-01-09:** Constrained decoding moved to GA with expanded model support. Parallel tool calling with constrained decoding added.
- **2026-01-06:** Z.ai GLM 4.7 preview added.
- **2025-12-18:** Batch API and Files API launched. Up to 50,000 requests per batch with 50% cost savings.
- **2025-12-17:** Parallel tool calling enabled for all models.
- **2025-12-16:** Streaming supported for all models including structured outputs. `reasoning_format` parameter added. Logprobs supported with structured outputs.
- **2025-12-10:** Prompt caching launched with automatic prefix reuse.
- **2025-11-24:** Predicted Outputs feature introduced.
- **2025-10-06:** Multi-token streaming introduced (200 events/s batches).
- **2025-10-02:** gpt-oss-120b expanded with tool calling `strict: true` and JSON schema response formats.
- **2025-08-13:** gpt-oss-120b moved to production.
- **2025-08-05:** gpt-oss-120b added as preview.
- **2024-10-24:** Speculative decoding implemented. Llama 3.1 70B at 2,100 tokens/s.
- **2024-10-03:** Performance improvements. Llama 3.1 8B at approximately 2,000 tokens/s. Microsoft AutoGen integration. `max_tokens` renamed to `max_completion_tokens`.

## Citations

- [1] Cerebras Inference Documentation - [https://inference-docs.cerebras.ai/](https://inference-docs.cerebras.ai/)
- [2] Cerebras API Reference: Chat Completions - [https://inference-docs.cerebras.ai/api-reference/chat-completions](https://inference-docs.cerebras.ai/api-reference/chat-completions)
- [3] Cerebras Supported Models - [https://inference-docs.cerebras.ai/models/overview](https://inference-docs.cerebras.ai/models/overview)
- [4] Cerebras Reasoning Capabilities - [https://inference-docs.cerebras.ai/capabilities/reasoning](https://inference-docs.cerebras.ai/capabilities/reasoning)
- [5] CePO: Cerebras Planning & Optimization - [https://inference-docs.cerebras.ai/capabilities/cepo](https://inference-docs.cerebras.ai/capabilities/cepo)
- [6] Cerebras Prompt Caching - [https://inference-docs.cerebras.ai/capabilities/prompt-caching](https://inference-docs.cerebras.ai/capabilities/prompt-caching)
- [7] Cerebras Service Tiers - [https://inference-docs.cerebras.ai/capabilities/service-tiers](https://inference-docs.cerebras.ai/capabilities/service-tiers)
- [8] Cerebras Rate Limits - [https://inference-docs.cerebras.ai/support/rate-limits](https://inference-docs.cerebras.ai/support/rate-limits)
- [9] Cerebras Change Log - [https://inference-docs.cerebras.ai/support/change-log](https://inference-docs.cerebras.ai/support/change-log)
- [10] Cerebras OpenAI Compatibility - [https://inference-docs.cerebras.ai/resources/openai](https://inference-docs.cerebras.ai/resources/openai)
- [11] Cerebras Integrations - [https://inference-docs.cerebras.ai/integrations](https://inference-docs.cerebras.ai/integrations)
- [12] Cerebras Error Codes - [https://inference-docs.cerebras.ai/support/error](https://inference-docs.cerebras.ai/support/error)
- [13] Cerebras Deprecations - [https://inference-docs.cerebras.ai/support/deprecation](https://inference-docs.cerebras.ai/support/deprecation)
- [14] Cerebras Metrics API - [https://inference-docs.cerebras.ai/capabilities/metrics](https://inference-docs.cerebras.ai/capabilities/metrics)
- [15] Cerebras API Reference: Completions - [https://inference-docs.cerebras.ai/api-reference/completions](https://inference-docs.cerebras.ai/api-reference/completions)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

Cerebras, wafer-scale engine, WSE, CS-3, hosted inference API, OpenAI-compatible, gpt-oss-120b, llama3.1-8b, Qwen 3 235B, Z.ai GLM 4.7, CePO, Cerebras Planning & Optimization, OptiLLM, prompt caching, service tiers, reasoning effort, reasoning format, structured outputs, parallel tool calls, predicted outputs, Batch API, Files API, Metrics API, speculative decoding, constrained decoding, fast inference, LPU alternative

### Verb-Noun Tasks

- Call the Cerebras chat completions endpoint with the Python SDK
- Swap an OpenAI base URL for `https://api.cerebras.ai/v1` to migrate code
- Stream tokens from gpt-oss-120b via Server-Sent Events
- Enforce a JSON schema on model output with `response_format`
- Issue parallel tool calls with constrained decoding
- Tune `reasoning_effort` (low/medium/high) on gpt-oss-120b
- Hide chain-of-thought tokens with `reasoning_format=hidden`
- Submit up to 50,000 requests through the Batch API at 50% cost savings
- Cache common prompt prefixes in 128-token blocks
- Run CePO planning/best-of-N selection on Llama via OptiLLM
- Route traffic with `service_tier` (priority/default/auto/flex)
- Monitor dedicated endpoints with the Prometheus-compatible Metrics API
- Accelerate generation with the `prediction` parameter

### User Intent Phrases

- "I want the fastest possible LLM token throughput"
- "How do I get 3,000 tokens/sec on a 120B-parameter model?"
- "What's the easiest way to migrate my OpenAI code to faster hardware?"
- "I need a hosted reasoning model with adjustable thinking depth"
- "How do I run gpt-oss-120b without managing GPUs?"
- "I need batch inference at half the cost for overnight jobs"
- "Show me an OpenAI-compatible endpoint that supports constrained JSON output"
- "How do I cache long system prompts to cut latency?"
- "I want parallel function calling on a hosted model"
- "Which inference API runs on wafer-scale chips instead of GPUs?"
- "How do I use CePO test-time compute with Llama?"

### Problem Statements

- GPU-based inference is too slow for real-time voice or chat UX
- OpenAI rate limits and latency block high-throughput agent loops
- Self-hosting large models requires GPU clusters and DevOps overhead
- Reasoning models burn tokens on visible chain-of-thought you don't need
- Repeated system prompts re-pay full prompt compute on every request
- Bulk inference jobs at on-demand pricing are too expensive
- Tool-calling JSON output is unreliable without schema enforcement

### When to Pick This

- Pick this when raw tokens-per-second on large models is the bottleneck
- Pick this over Groq when you need the largest open-weight models (120B+) at hosted speed
- Pick this when you want OpenAI API compatibility with no model self-hosting
- Pick this when adjustable reasoning effort matters for cost-accuracy tuning
- Pick this when batch workloads can tolerate async processing for 50% savings
- Pick this when prompt caching on repeated long prefixes will dominate latency
- Pick this when CePO-style test-time compute on Llama is desired
- Skip this when you need fine-tuning, custom models, or self-hosted deployment

### Related Terms and Aliases

- Cerebras Cloud, Cerebras Inference, Cerebras Systems
- CS-3, WSE-3, wafer-scale processor
- "Fast inference API", "high-throughput LLM API"
- CePO, Cerebras Planning and Optimization
- OptiLLM, test-time compute
- Speculative decoding, multi-token streaming
- Alternative to Groq, Together AI, Fireworks AI, SambaNova
