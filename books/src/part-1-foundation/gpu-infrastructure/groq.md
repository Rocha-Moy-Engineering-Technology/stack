# Groq

> Hosted API for fast LLM inference on custom LPU hardware

| Field | Value |
|-------|-------|
| Name | Groq |
| Group | GPU Infrastructure |
| Type | API/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Docs | [console.groq.com/docs](https://console.groq.com/docs/overview) |

## Overview

Groq is an inference platform built on custom Language Processing Unit (LPU) hardware, designed to deliver fast Large Language Model (LLM) inference through an OpenAI-compatible API. Unlike GPU-based inference providers, Groq uses purpose-built Application-Specific Integrated Circuit (ASIC) silicon optimized for sequential token generation, producing deterministic low-latency inference. The platform exposes a REST API at `https://api.groq.com/openai/v1` and provides official Software Development Kits (SDKs) for Python and JavaScript/TypeScript. Groq hosts a curated set of open-weight models spanning text generation, reasoning, speech-to-text, text-to-speech, vision, and content moderation. The platform also offers agentic AI systems (Compound and Compound Mini) with built-in tool orchestration, a Responses API for advanced agentic workflows, and Model Context Protocol (MCP) support for connecting to external tool servers. [1]

## Core Concepts

- **Language Processing Unit (LPU)**: Groq's custom ASIC hardware architecture, purpose-built for sequential inference workloads rather than the parallel matrix operations GPUs are optimized for. The LPU architecture delivers deterministic, low-latency token generation. Models reside in LPU memory continuously, eliminating cold-start latency.
- **OpenAI-Compatible API**: Groq's API follows the OpenAI chat completions interface, meaning existing code targeting the OpenAI SDK can be redirected to Groq by changing the base URL and API key with minimal modification. Known incompatibilities include lack of support for `logprobs`, `logit_bias`, `top_logprobs`, `messages[].name`, and the N parameter (must equal 1). A temperature value of 0 is converted to `1e-8`. [12]
- **Service Tiers**: Groq offers three processing tiers -- Performance Tier with dedicated compute resources and guaranteed availability, Flex Processing for cost-optimized high-throughput workloads with 10x higher rate limits but no availability guarantee, and Batch Processing for asynchronous bulk inference at 50% lower cost with 24-hour to 7-day completion windows. [6][8]
- **Prompt Caching**: Groq automatically caches prompt prefixes from recent requests. When a subsequent request shares the same prefix, cached computation is reused, reducing both latency and cost by 50% for cached token portions. Cached tokens do not count toward rate limits. Caches expire after 2 hours without use. Minimum cacheable prompt length varies by model (128 to 1024 tokens). [7]
- **Groq Compound**: Agentic AI systems (Compound and Compound Mini) with built-in tools including web search, code execution, Wolfram Alpha integration, and parallel browser automation (up to 10 pages simultaneously). These handle tool orchestration autonomously in a single API call. [14]
- **Responses API (Beta)**: An OpenAI-compatible Responses API supporting text and image inputs, function calling, built-in tools, MCP integration, structured outputs, and reasoning. Does not yet support stateful conversations (`previous_response_id` is unavailable). [10]
- **Model Context Protocol (MCP)**: Server-side remote tool calling via MCP servers. Groq discovers tools from MCP servers, passes definitions to the model, executes tool calls, and returns results -- all within a single API request. [11]
- **LoRA Inference**: Enterprise-only support for Low-Rank Adaptation (LoRA) adapters, allowing serving fine-tuned model variants without hosting separate full model copies. Adapters must be trained externally and uploaded to Groq. Currently limited to `llama-3.1-8b-instant` base model. [9]

## Installation

### Python SDK

```bash
pip install groq
```

### JavaScript/TypeScript SDK

```bash
npm install groq-sdk
```

### Authentication

Groq uses API key authentication via the `GROQ_API_KEY` environment variable:

```bash
export GROQ_API_KEY="gsk_your_api_key_here"
```

The API key is obtained from the Groq Console at `https://console.groq.com`. Both SDKs automatically read `GROQ_API_KEY` from the environment when no key is explicitly passed to the client constructor. [1]

## Architecture

Groq's architecture consists of three layers:

1. **Hardware Layer**: Custom LPU chips arranged in GroqRack systems. Each LPU handles inference deterministically, meaning the same input produces identical timing characteristics across runs. This contrasts with GPU inference, where batching and scheduling introduce variable latency.
2. **API Gateway Layer**: An OpenAI-compatible REST API that routes requests to model-specific inference endpoints. The gateway handles authentication, rate limiting, prompt caching, service tier selection, and MCP tool orchestration.
3. **Model Serving Layer**: Pre-loaded open-weight models served directly from LPU memory. Models are not loaded on-demand; they reside in hardware memory continuously, eliminating cold-start latency. Inference speeds range from 200 to 1,000+ tokens per second depending on model size.

## Key Features

- **Text Generation (Chat Completions)**: Standard chat completions endpoint supporting streaming, asynchronous calls, stop sequences, temperature control (0.0 to 2.0, default 0.5), top-p sampling, and max completion tokens. [2]
- **Reasoning**: Dedicated reasoning capabilities via GPT-OSS models (with `include_reasoning` parameter and `low`/`medium`/`high` effort levels) and Qwen3-32B (with `reasoning_format` parameter supporting `parsed`, `raw`, or `hidden` modes). Recommended temperature 0.5-0.7. Cannot use `raw` format with JSON mode or tool use. [4]
- **Speech-to-Text**: Transcription and translation via Whisper model variants. Supports FLAC, MP3, MP4, MPEG, MPGA, M4A, OGG, WAV, and WebM formats. File size limits: 25 MB (free tier), 100 MB (dev tier). Audio is downsampled to 16KHz mono. Response formats include JSON, verbose_json (with timestamps and quality metadata), and plain text. [5]
- **Text-to-Speech**: Audio generation via Orpheus models (`canopylabs/orpheus-v1-english` and `canopylabs/orpheus-arabic-saudi`) with vocal direction controls (e.g., `[cheerful]` tags). Output defaults to WAV format. [13]
- **Vision**: Image analysis through multimodal models (Llama 4 Scout 17B). Supports URL-based images (up to 20 MB) and base64-encoded images (up to 4 MB). Maximum 5 images per request, 33 megapixel resolution ceiling. Includes Optical Character Recognition (OCR) capabilities. [3]
- **Tool Use**: Function calling support with three patterns -- built-in tools (web search, code execution, browser automation, Wolfram Alpha) executed on Groq infrastructure; remote MCP tools via third-party servers; and local tool calling with custom function definitions. Supports parallel tool calls on most models. [15]
- **Structured Outputs**: Two modes -- Strict mode (`strict: true`) with constrained decoding guaranteeing 100% schema adherence (GPT-OSS models only), and Best-effort mode (`strict: false`) available across more models with retry-based validation. Also supports basic JSON Object Mode for models without full structured output support. [16]
- **Content Moderation**: GPT-OSS-Safeguard 20B for bring-your-own-policy trust and safety workflows with reasoning explanations. Llama Prompt Guard 2 (22M and 86M parameter variants) for prompt injection detection. Llama Guard 4 12B for multimodal content moderation using MLCommons Taxonomy. [17]
- **Batch Processing**: Asynchronous batch API supporting chat completions, audio transcription, and audio translation. Up to 50,000 requests per file, 200 MB maximum. 50% cost discount versus synchronous pricing. Completion windows from 24 hours to 7 days. Results retained for 30 days. [8]

## Use Cases

- **Low-Latency Chatbots**: Applications requiring sub-second response times for interactive conversation, where Groq's LPU latency advantage over GPU inference is most pronounced. At 300-1,000+ tokens per second, multi-turn conversations feel instantaneous.
- **Agentic Workflows**: Tool-augmented agents that combine text generation with function calling (web search, code execution, MCP tools). Fast inference reduces end-to-end agent loop time -- a typical multi-tool workflow requiring 3-5 inference calls completes in seconds rather than minutes.
- **Real-Time Speech Processing**: Transcription pipelines using Whisper models at 189-216x real-time speed factor for live audio streams or recorded media.
- **High-Throughput Document Processing**: Batch processing tier for summarization, extraction, or classification across large document corpora at 50% reduced cost.
- **Content Moderation Pipelines**: Automated safety screening with custom policies using GPT-OSS-Safeguard, prompt injection detection via Llama Prompt Guard, or multimodal content analysis with Llama Guard 4.
- **Structured Data Extraction**: Vision OCR combined with structured outputs for extracting typed data from documents and images with guaranteed schema compliance.

## API Reference

### Endpoints

**Chat Completions**: `POST /openai/v1/chat/completions` -- Creates a model response for a chat conversation. [2]

**Responses (Beta)**: `POST /openai/v1/responses` -- Advanced API for agentic workflows with built-in tool support and MCP integration. [10]

**Transcription**: `POST /openai/v1/audio/transcriptions` -- Transcribes audio into the input language. [5]

**Translation**: `POST /openai/v1/audio/translations` -- Translates audio into English. [5]

**Speech**: `POST /openai/v1/audio/speech` -- Generates audio from input text. [13]

**Models**: `GET /openai/v1/models` -- Lists available models. `GET /openai/v1/models/{model}` -- Retrieves model details.

**Batches**: `POST /openai/v1/batches` -- Creates a batch job. `GET /openai/v1/batches/{batch_id}` -- Retrieves batch status. `POST /openai/v1/batches/{batch_id}/cancel` -- Cancels a batch. [8]

**Files**: `POST /openai/v1/files` -- Uploads a file (100 MB max, JSONL). `GET /openai/v1/files` -- Lists files. `GET /openai/v1/files/{file_id}/content` -- Downloads file content.

**Fine-Tuning (Beta)**: `POST /v1/fine_tunings` -- Registers a LoRA adapter. `GET /v1/fine_tunings` -- Lists adapters. Enterprise only. [9]

### Chat Completions Parameters

- `messages` (required): Array of message objects with `role` (system, user, assistant, tool) and `content` fields.
- `model` (required): Model identifier string (e.g., `llama-3.3-70b-versatile`).
- `temperature`: Sampling temperature, default `0.5`. Range `0.0` to `2.0`.
- `max_completion_tokens`: Maximum tokens in the generated response.
- `top_p`: Nucleus sampling threshold.
- `stop`: String or array of strings where the model stops generating.
- `stream`: Boolean to enable Server-Sent Events (SSE) streaming of partial responses.
- `tools`: Array of tool definitions in JSON Schema format for function calling.
- `response_format`: Structured output specification (`json_schema` or `json_object`).
- `service_tier`: Processing tier selection (`flex` for high-throughput).

### Available Models

**Text Generation (Production)**:
- `llama-3.1-8b-instant` -- 8B parameter Llama 3.1, 131,072 context window, 560 tps, $0.05/$0.08 per million tokens
- `llama-3.3-70b-versatile` -- 70B parameter Llama 3.3, 131,072 context window, 280 tps, $0.59/$0.79 per million tokens
- `openai/gpt-oss-20b` -- 20B parameter GPT-OSS, 131,072 context window, 1,000 tps, $0.075/$0.30 per million tokens
- `openai/gpt-oss-120b` -- 120B parameter GPT-OSS, 131,072 context window, 500+ tps
- `openai/gpt-oss-safeguard-20b` -- 20B parameter safety model, 131,072 context window, ~1,000 tps

**Agentic Systems**:
- `groq/compound` -- Agentic AI with built-in tools, 131,072 context, 8,192 max completion, ~450 tps
- `groq/compound-mini` -- Lightweight agentic AI, 131,072 context, 8,192 max completion, ~450 tps

**Speech-to-Text**:
- `whisper-large-v3` -- Full Whisper v3, $0.111/hour, 10.3% word error rate, 189x real-time, supports translation
- `whisper-large-v3-turbo` -- Optimized Whisper v3, $0.04/hour, 12% word error rate, 216x real-time

**Text-to-Speech**:
- `canopylabs/orpheus-v1-english` -- Expressive English TTS with vocal direction controls
- `canopylabs/orpheus-arabic-saudi` -- Saudi Arabic dialect synthesis

**Safety**:
- `llama-guard-4-12b` -- Multimodal content moderation, 128K context
- `meta-llama/llama-prompt-guard-2-86m` -- Prompt injection detection (86M params)
- `meta-llama/llama-prompt-guard-2-22m` -- Prompt injection detection (22M params)

**Preview Models** (evaluation only, may be discontinued without notice):
- `meta-llama/llama-4-scout-17b-16e-instruct` -- 17B x 16 expert MoE, 750 tps, vision support
- `moonshotai/kimi-k2-instruct-0905` -- 262,144 context, 200 tps, $1.00/$3.00 per million tokens
- `qwen/qwen3-32b` -- 128K context, 400 tps, reasoning support, $0.29/$0.59 per million tokens

## Configuration

### Client Configuration (Python)

```python
from groq import Groq

# Reads GROQ_API_KEY from environment automatically
client = Groq()

# Explicit configuration
client = Groq(
    api_key="gsk_your_api_key_here",
    base_url="https://api.groq.com/openai/v1"
)

# Async client
from groq import AsyncGroq
async_client = AsyncGroq()
```

### Client Configuration (JavaScript/TypeScript)

```typescript
import Groq from "groq-sdk";

// Reads GROQ_API_KEY from environment automatically
const client = new Groq();

// Explicit configuration
const client = new Groq({
    apiKey: "gsk_your_api_key_here"
});
```

### Environment Variables

- `GROQ_API_KEY`: Required. API key for authentication obtained from `https://console.groq.com`.

### Rate Limits

Rate limits are enforced at the organization level across six dimensions: requests per minute (RPM), requests per day (RPD), tokens per minute (TPM), tokens per day (TPD), audio seconds per hour (ASH), and audio seconds per day (ASD). The API returns HTTP 429 when any limit is exceeded, with `retry-after` and `x-ratelimit-remaining-*` headers. Cached tokens do not count toward rate limits. [6]

**Free tier examples**:
- `llama-3.1-8b-instant`: 30 RPM, 14,400 RPD, 6,000 TPM, 500,000 TPD
- `llama-3.3-70b-versatile`: 30 RPM, 1,000 RPD, 12,000 TPM, 100,000 TPD
- `whisper-large-v3`: 20 RPM, 2,000 RPD

Higher limits are available on the Developer plan and for enterprise workloads.

### Inference Metrics

To include detailed performance metrics in API responses, set the header `Groq-Beta: inference-metrics`. Response metadata includes completion time, prompt processing time, queue time, and total request duration. [10]

## Integration Patterns

### OpenAI SDK Compatibility

Groq can be used as a drop-in replacement with the OpenAI Python SDK by overriding the base URL:

```python
from openai import OpenAI

client = OpenAI(
    api_key="gsk_your_api_key_here",
    base_url="https://api.groq.com/openai/v1"
)

response = client.chat.completions.create(
    messages=[{"role": "user", "content": "Hello"}],
    model="llama-3.3-70b-versatile"
)
```

Unsupported OpenAI parameters: `logprobs`, `logit_bias`, `top_logprobs`, `messages[].name`, N > 1, and `vtt`/`srt` text completion formats. [12]

### REST/curl

```bash
curl -X POST "https://api.groq.com/openai/v1/chat/completions" \
  -H "Authorization: Bearer $GROQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "Hello"}],
    "model": "llama-3.3-70b-versatile"
  }'
```

### Flex Processing

Add `"service_tier": "flex"` to the request body for 10x higher rate limits (paid plans only). Requests may fail with HTTP 498 when capacity is unavailable; implement jittered backoff and retries. [6]

### Prompt Caching Optimization

Place static content (system prompts, tool definitions, few-shot examples) at the beginning of messages and dynamic content (user queries, session data) at the end to maximize cache hit rate. Monitor cache performance via the `prompt_tokens_details.cached_tokens` field in API responses. [7]

### Remote MCP Integration

```javascript
const response = await client.responses.create({
    model: "openai/gpt-oss-120b",
    input: "What models are trending on Huggingface?",
    tools: [{
        type: "mcp",
        server_label: "Huggingface",
        server_url: "https://huggingface.co/mcp",
        require_approval: "never"
    }]
});
```

Multiple MCP servers can be combined in a single request. Only connect to trusted servers, as MCP servers have access to all data in the model's context including messages, system prompts, and conversation history. [11]

## Examples

### Basic Chat Completion

```python
from groq import Groq

client = Groq()
chat_completion = client.chat.completions.create(
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain the importance of fast language models"}
    ],
    model="llama-3.3-70b-versatile"
)
print(chat_completion.choices[0].message.content)
```

### Streaming Response

```python
from groq import Groq

client = Groq()
stream = client.chat.completions.create(
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum computing in simple terms."}
    ],
    model="llama-3.3-70b-versatile",
    temperature=0.5,
    max_completion_tokens=1024,
    top_p=1,
    stream=True
)
for chunk in stream:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end="")
```

### Async Chat Completion

```python
import asyncio
from groq import AsyncGroq

async def main():
    client = AsyncGroq()
    chat_completion = await client.chat.completions.create(
        messages=[
            {"role": "user", "content": "Explain the importance of fast language models"}
        ],
        model="llama-3.3-70b-versatile"
    )
    print(chat_completion.choices[0].message.content)

asyncio.run(main())
```

### Structured Output with JSON Schema (Strict Mode)

```python
from groq import Groq

client = Groq()
response = client.chat.completions.create(
    messages=[{"role": "user", "content": "List three programming languages with their use cases."}],
    model="openai/gpt-oss-20b",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "languages",
            "strict": True,
            "schema": {
                "type": "object",
                "properties": {
                    "languages": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "name": {"type": "string"},
                                "use_case": {"type": "string"}
                            },
                            "required": ["name", "use_case"],
                            "additionalProperties": False
                        }
                    }
                },
                "required": ["languages"],
                "additionalProperties": False
            }
        }
    }
)
```

### Tool Use (Function Calling)

```python
from groq import Groq
import json

client = Groq()

tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get current weather for a location",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "City and state"},
                "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
            },
            "required": ["location"]
        }
    }
}]

response = client.chat.completions.create(
    messages=[{"role": "user", "content": "What's the weather in San Francisco?"}],
    model="llama-3.3-70b-versatile",
    tools=tools,
    tool_choice="auto"
)

# Handle tool call
tool_call = response.choices[0].message.tool_calls[0]
# Execute function, then send result back
follow_up = client.chat.completions.create(
    messages=[
        {"role": "user", "content": "What's the weather in San Francisco?"},
        response.choices[0].message,
        {
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": json.dumps({"temperature": 72, "condition": "sunny"})
        }
    ],
    model="llama-3.3-70b-versatile",
    tools=tools
)
```

### Speech-to-Text Transcription

```python
from groq import Groq

client = Groq()
with open("audio.mp3", "rb") as audio_file:
    transcription = client.audio.transcriptions.create(
        file=audio_file,
        model="whisper-large-v3-turbo",
        response_format="verbose_json",
        timestamp_granularities=["word", "segment"]
    )
print(transcription.text)
```

### Reasoning with GPT-OSS

```python
from groq import Groq

client = Groq()
response = client.chat.completions.create(
    messages=[{"role": "user", "content": "Solve this step by step: If a train travels 120km in 2 hours, then slows to cover 80km in 2 more hours, what is the average speed for the entire trip?"}],
    model="openai/gpt-oss-20b",
    reasoning_effort="high",
    include_reasoning=True,
    temperature=0.6
)
```

### Batch Processing

```python
from groq import Groq

client = Groq()

# Step 1: Upload JSONL file
with open("batch_requests.jsonl", "rb") as f:
    file = client.files.create(file=f, purpose="batch")

# Step 2: Create batch
batch = client.batches.create(
    input_file_id=file.id,
    endpoint="/v1/chat/completions",
    completion_window="24h"
)

# Step 3: Poll for completion
import time
while batch.status not in ("completed", "failed", "expired"):
    time.sleep(30)
    batch = client.batches.retrieve(batch.id)

# Step 4: Download results
if batch.output_file_id:
    content = client.files.content(batch.output_file_id)
```

### JavaScript Streaming

```javascript
import Groq from "groq-sdk";

const client = new Groq();

const stream = await client.chat.completions.create({
    messages: [
        {role: "system", content: "You are a helpful assistant."},
        {role: "user", content: "Explain the importance of fast language models"}
    ],
    model: "llama-3.3-70b-versatile",
    temperature: 0.5,
    max_completion_tokens: 1024,
    stream: true
});

for await (const chunk of stream) {
    process.stdout.write(chunk.choices[0]?.delta?.content || "");
}
```

## Limitations

- **Model Selection**: Groq hosts a curated subset of open-weight models. Custom model uploads or arbitrary model hosting is not supported outside of enterprise-only LoRA adapters.
- **Closed-Source Hardware**: The LPU architecture is proprietary. There is no self-hosted or on-premises deployment option; all inference runs on Groq's managed infrastructure.
- **Rate Limits**: Each tier has distinct rate limits. Free tier limits are restrictive (e.g., 30 RPM, 1,000 RPD for 70B models). Higher limits require paid plans. Flex tier provides 10x limits but without availability guarantees.
- **Context Window Constraints**: Maximum context windows vary by model, with most production models at 131,072 tokens. Context windows are fixed per model and cannot be extended.
- **No Fine-Tuning Service**: Groq does not offer a fine-tuning API. LoRA adapters must be trained externally, are enterprise-only, and currently support only `llama-3.1-8b-instant` as a base model with ranks limited to 8, 16, 32, or 64.
- **Responses API Limitations**: The beta Responses API does not yet support stateful conversations (`previous_response_id`), `store`, `truncation`, or `prompt_cache_key`.
- **Structured Outputs Constraints**: Strict mode (guaranteed schema adherence) is available only on GPT-OSS models. Streaming and tool use are unsupported with structured outputs.
- **OpenAI SDK Gaps**: Several OpenAI parameters are unsupported: `logprobs`, `logit_bias`, `top_logprobs`, `messages[].name`, N > 1, and `vtt`/`srt` formats.
- **Reasoning Restrictions**: Cannot use `raw` reasoning format when JSON mode or tool use are enabled. System prompts should be avoided with reasoning models.
- **Regional Availability**: Infrastructure is concentrated in specific data center regions. LoRA inference is not available for regional/sovereign endpoints.

## Changelog

- **December 2025**: MCP Connectors (Beta) for Google Workspace (Gmail, Calendar, Drive) with OAuth 2.0 authentication. [14]
- **October 2025**: GPT-OSS-Safeguard 20B safety model with bring-your-own-policy content moderation at ~1,000 tps. Prompt caching extended to GPT-OSS 120B. SDK updates: Python v0.33.0, TypeScript v0.34.0. [14]
- **September 2025**: Remote MCP support (Beta) for external tool servers. Groq Compound and Compound Mini reach General Availability with web search, code execution, Wolfram Alpha, and parallel browser automation. Kimi K2 Instruct with 256K context. Prompt caching launched for GPT-OSS 20B and Kimi K2. [14]
- **August 2025**: GPT-OSS 20B (1,000+ tps) and GPT-OSS 120B (500+ tps) launched. Responses API (Beta) introduced. Automatic prompt caching feature launched. [14]
- **July 2025**: Structured Outputs with JSON Schema support. Kimi 2 Instruct (1T parameter MoE). [14]
- **June 2025**: Qwen3-32B with reasoning support and 128K context. SDK reasoning field additions. [14]
- **May 2025**: Llama Prompt Guard 2 (22M/86M) for prompt injection detection. Llama Guard 4 12B multimodal moderation. Compound Beta search settings with domain filtering. [14]
- **April 2025**: Llama 4 Scout and Maverick models with vision support. Compound Beta and Compound Beta Mini agentic systems. Gemma-7b-it and Mixtral-8x7b-32768 deprecated. [14]

## Citations

- [1] Groq Documentation Overview - https://console.groq.com/docs/overview
- [2] Groq Text Chat API - https://console.groq.com/docs/text-chat
- [3] Groq Vision - https://console.groq.com/docs/vision
- [4] Groq Reasoning - https://console.groq.com/docs/reasoning
- [5] Groq Speech-to-Text - https://console.groq.com/docs/speech-to-text
- [6] Groq Rate Limits - https://console.groq.com/docs/rate-limits
- [7] Groq Prompt Caching - https://console.groq.com/docs/prompt-caching
- [8] Groq Batch Processing - https://console.groq.com/docs/batch
- [9] Groq LoRA Inference - https://console.groq.com/docs/lora
- [10] Groq Responses API - https://console.groq.com/docs/responses-api
- [11] Groq MCP Support - https://console.groq.com/docs/mcp
- [12] Groq OpenAI Compatibility - https://console.groq.com/docs/openai
- [13] Groq Text-to-Speech - https://console.groq.com/docs/text-to-speech
- [14] Groq Changelog - https://console.groq.com/docs/changelog
- [15] Groq Tool Use - https://console.groq.com/docs/tool-use
- [16] Groq Structured Outputs - https://console.groq.com/docs/structured-outputs
- [17] Groq Content Moderation - https://console.groq.com/docs/content-moderation
- [18] Groq API Reference - https://console.groq.com/docs/api-reference
- [19] Groq Flex Processing - https://console.groq.com/docs/flex-processing
- [20] Groq Error Codes - https://console.groq.com/docs/errors
