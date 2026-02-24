# Groq

| Field          | Value                                                        |
|----------------|--------------------------------------------------------------|
| **Group**      | GPU Compute & Cloud Platforms                                |
| **Type**       | API/Infra                                                    |
| **Open Source** | No                                                          |
| **GitHub**     | N/A                                                          |
| **Stars**      | N/A                                                          |
| **Docs**       | [Official Docs](https://console.groq.com/docs/overview)      |

## Overview

Groq is an inference platform built on custom Language Processing Unit (LPU) hardware, designed to deliver fast Large Language Model (LLM) inference through an OpenAI-compatible API. Unlike GPU-based inference providers, Groq uses purpose-built silicon optimized for sequential token generation, which results in significantly lower latency per token. The platform exposes a REST API at `https://api.groq.com/openai/v1` and provides official Software Development Kits (SDKs) for Python and JavaScript/TypeScript. Groq hosts a curated set of open-weight models spanning text generation, speech-to-text, vision, and content moderation. [1]

## Core Concepts

- **Language Processing Unit (LPU)**: Groq's custom Application-Specific Integrated Circuit (ASIC) hardware architecture, purpose-built for sequential inference workloads rather than the parallel matrix operations GPUs are optimized for. The LPU architecture delivers deterministic, low-latency token generation.
- **OpenAI-Compatible API**: Groq's API follows the OpenAI chat completions interface, meaning existing code targeting the OpenAI SDK can be redirected to Groq by changing the base URL and API key with minimal modification.
- **Service Tiers**: Groq offers three processing tiers -- Performance Tier with dedicated compute resources, Flex Processing for cost-optimized workloads, and Batch Processing for asynchronous bulk inference jobs.
- **Prompt Caching**: Groq caches prompt prefixes so that repeated requests sharing the same system prompt or conversation prefix skip redundant computation, reducing both latency and cost.
- **LoRA Inference**: Support for Low-Rank Adaptation (LoRA) adapters allows serving fine-tuned model variants without hosting separate full model copies.

## Installation and Setup

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

The API key is obtained from the Groq Console at `https://console.groq.com`. The Python and JavaScript SDKs automatically read `GROQ_API_KEY` from the environment when no key is explicitly passed to the client constructor. [1]

## Architecture

Groq's architecture consists of three layers:

1. **Hardware Layer**: Custom LPU chips arranged in GroqRack systems. Each LPU handles inference deterministically, meaning the same input produces identical timing characteristics across runs. This contrasts with GPU inference, where batching and scheduling introduce variable latency.
2. **API Gateway Layer**: An OpenAI-compatible REST API that routes requests to model-specific inference endpoints. The gateway handles authentication, rate limiting, prompt caching, and service tier selection.
3. **Model Serving Layer**: Pre-loaded open-weight models (Llama, Gemma, Whisper, and others) served directly from LPU memory. Models are not loaded on-demand; they reside in hardware memory continuously, eliminating cold-start latency.

## Key Features

- **Text Generation (Chat Completions)**: Standard chat completions endpoint supporting streaming, asynchronous calls, stop sequences, temperature control, and top-p sampling. [2]
- **Speech-to-Text**: Transcription and translation via Whisper model variants (whisper-large-v3, whisper-large-v3-turbo, distil-whisper-large-v3-en).
- **Vision**: Optical Character Recognition (OCR) and image recognition capabilities through multimodal model endpoints.
- **Tool Use**: Function calling support including web search, browser automation, code execution, and Wolfram Alpha integration.
- **Reasoning**: Dedicated reasoning capabilities for multi-step problem solving.
- **Structured Outputs**: JSON schema validation on model outputs, ensuring responses conform to a developer-specified schema.
- **Content Moderation**: Safety classification via dedicated models such as llama-guard-3-8b.
- **Batch Processing**: Asynchronous batch API for submitting large volumes of requests to be processed without real-time latency requirements.

## Use Cases

- **Low-Latency Chatbots**: Applications requiring sub-second response times for interactive conversation, where Groq's LPU latency advantage over GPU inference is most pronounced.
- **Real-Time Speech Processing**: Transcription pipelines using Whisper models for live audio streams or recorded media.
- **High-Throughput Document Processing**: Batch processing tier for summarization, extraction, or classification across large document corpora.
- **Tool-Augmented Agents**: Agentic workflows that combine text generation with tool use (web search, code execution) where fast inference reduces end-to-end agent loop time.
- **Content Moderation Pipelines**: Automated safety screening of user-generated content using llama-guard-3-8b before publishing or further processing.

## API Reference Summary

### Chat Completions

**Endpoint**: `POST /openai/v1/chat/completions`

**Parameters**:
- `messages`: Array of message objects with `role` (system, user, assistant) and `content` fields.
- `model`: Model identifier string (e.g., `llama-3.3-70b-versatile`).
- `temperature`: Sampling temperature, default `0.5`. Range `0.0` to `2.0`.
- `max_completion_tokens`: Maximum tokens in the generated response.
- `top_p`: Nucleus sampling threshold.
- `stop`: String or array of strings where the model stops generating.
- `stream`: Boolean to enable Server-Sent Events (SSE) streaming of partial responses.

### Available Models

**Text Generation**:
- `llama3-8b-8192` -- 8B parameter Llama 3, 8192 context window
- `llama3-70b-8192` -- 70B parameter Llama 3, 8192 context window
- `llama-3.1-8b-instant` -- 8B parameter Llama 3.1, 131072 context window
- `llama-3.3-70b-versatile` -- 70B parameter Llama 3.3, general purpose
- `gemma2-9b-it` -- 9B parameter Gemma 2 instruction-tuned, 8192 context window
- `openai/gpt-oss-20b` -- 20B parameter open-source GPT variant

**Speech-to-Text**:
- `whisper-large-v3` -- Full Whisper v3
- `whisper-large-v3-turbo` -- Optimized Whisper v3
- `distil-whisper-large-v3-en` -- Distilled English-only Whisper v3

**Safety**:
- `llama-guard-3-8b` -- Content moderation classifier

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

- `GROQ_API_KEY`: Required. API key for authentication.

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

## Examples

### Basic Chat Completion

```python
from groq import Groq

client = Groq()
chat_completion = client.chat.completions.create(
    messages=[{"role": "user", "content": "Hello"}],
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
    stream=True
)
for chunk in stream:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end="")
```

### Structured Output with JSON Schema

```python
from groq import Groq

client = Groq()
response = client.chat.completions.create(
    messages=[{"role": "user", "content": "List three programming languages."}],
    model="llama-3.3-70b-versatile",
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "languages",
            "schema": {
                "type": "object",
                "properties": {
                    "languages": {
                        "type": "array",
                        "items": {"type": "string"}
                    }
                },
                "required": ["languages"]
            }
        }
    }
)
```

### Speech-to-Text Transcription

```python
from groq import Groq

client = Groq()
with open("audio.mp3", "rb") as audio_file:
    transcription = client.audio.transcriptions.create(
        file=audio_file,
        model="whisper-large-v3-turbo"
    )
print(transcription.text)
```

## Limitations

- **Model Selection**: Groq hosts a curated subset of open-weight models. Custom model uploads or arbitrary model hosting is not supported outside of LoRA adapters.
- **Closed-Source Hardware**: The LPU architecture is proprietary. There is no self-hosted or on-premises deployment option; all inference runs on Groq's managed infrastructure.
- **Rate Limits**: Each service tier has distinct rate limits on requests per minute and tokens per minute. The Performance Tier provides the highest throughput but at higher cost.
- **Context Window Constraints**: Maximum context windows vary by model, ranging from 8192 tokens (Llama 3 base, Gemma 2) to 131072 tokens (Llama 3.1 instant). These are fixed per model and cannot be extended.
- **No Fine-Tuning**: Groq does not offer a fine-tuning API. LoRA adapters must be trained externally and uploaded for inference.
- **Regional Availability**: Infrastructure is concentrated in specific data center regions, which may affect latency for geographically distant clients.

## Changelog Highlights

- Addition of Llama 3.3 70B Versatile model with improved general-purpose capabilities.
- Introduction of Batch Processing API for asynchronous bulk inference.
- Prompt caching support to reduce latency and cost for repeated prompt prefixes.
- LoRA inference support for serving fine-tuned model adapters.
- Structured output with JSON schema validation.
- Tool use expansion to include web search, browser automation, code execution, and Wolfram Alpha.
- Whisper model variants added for speech-to-text (full, turbo, distilled English).

## Citations

- [1] Groq Documentation - https://console.groq.com/docs/overview
- [2] Groq Text Chat API - https://console.groq.com/docs/text-chat
