# LM Studio

> Desktop app for running local LLMs with Python/TS SDKs and OpenAI-compatible endpoints

| Field | Value |
|-------|-------|
| Group | Inference Serving |
| Type | SDK/UI |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://lmstudio.ai/docs/) |

## Overview

LM Studio is a desktop application for downloading and running large language models locally on personal computers. It supports macOS, Windows, and Linux, running GGUF (via llama.cpp) and MLX format models without requiring cloud services. The platform provides a built-in chat interface, integrated model discovery from HuggingFace, an OpenAI-compatible REST API, an Anthropic-compatible messaging API, and a native REST API for programmatic access. [1]

LM Studio offers SDKs for TypeScript (lmstudio-js) and Python (lmstudio-python), a command-line interface (lms/llmster), and a headless daemon mode (llmster) for server deployments without the GUI. The latest version (0.4.1) introduced Anthropic-compatible endpoints and MCP server integration. [1]

## Core Concepts

### Local Model Execution

LM Studio runs models entirely on the local machine after download. Core functions -- chatting, document processing, and local server operation -- operate offline, keeping data private and local. [1]

### Model Formats

- **GGUF** -- Quantized model format for llama.cpp backend, supporting various quantization levels (Q4, Q5, Q8, FP16)
- **MLX** -- Apple's ML framework format optimized for Apple Silicon Macs [1]

### Local Server

LM Studio exposes a local HTTP server with multiple API compatibility layers (OpenAI, Anthropic, native), enabling integration with any tool that supports these APIs. [1]

### Model Discovery

Integrated HuggingFace browser for searching, filtering, and downloading models directly within the application. [1]

### MCP Integration

Model Context Protocol (MCP) server integration enables LM Studio to connect with external tools and data sources, extending model capabilities. [1]

## Architecture

### Application Components

- **Model Manager** -- Downloads, caches, and manages local model files
- **Inference Engine** -- llama.cpp (GGUF) and MLX (Apple Silicon) backends
- **Chat Interface** -- Built-in conversational UI with conversation history
- **Local Server** -- HTTP API server with OpenAI/Anthropic/native compatibility
- **Document Processor** -- Offline RAG for document attachment and querying
- **CLI (lms)** -- Command-line interface for model management and server control

### API Layers

LM Studio provides four API compatibility layers:

1. **OpenAI-compatible** -- `/v1/chat/completions`, `/v1/completions`, `/v1/embeddings`
2. **Anthropic-compatible** -- `POST /v1/messages` for Claude-style API
3. **Native REST** -- `/api/v1/` endpoints with LM Studio-specific features
4. **SDKs** -- TypeScript and Python native client libraries [1]

### Performance Features

- Speculative decoding for faster inference
- Continuous batching for parallel request handling
- Per-model default configuration presets
- GPU layer offloading control [1]

## Key Features and Functionality

### Chat Interface

Built-in chat UI with:

- Multi-chat split-view functionality
- Conversation history management
- Document attachment for offline RAG
- System prompt customization
- Generation parameter controls [1]

### OpenAI-Compatible Server

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")

response = client.chat.completions.create(
    model="lmstudio-community/Meta-Llama-3.1-8B-Instruct-GGUF",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"},
    ],
    temperature=0.7,
)
print(response.choices[0].message.content)
```
[1]

### Anthropic-Compatible Endpoint

```python
import anthropic

client = anthropic.Anthropic(base_url="http://localhost:1234/v1", api_key="lm-studio")

message = client.messages.create(
    model="lmstudio-community/Meta-Llama-3.1-8B-Instruct-GGUF",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello!"}],
)
```
[1]

### TypeScript SDK

```typescript
import { LMStudio } from "lmstudio-js";

const client = new LMStudio();
const model = await client.llm.load("lmstudio-community/Meta-Llama-3.1-8B-Instruct-GGUF");
const response = await model.respond([
    { role: "user", content: "Hello!" }
]);
```
[1]

### Python SDK

```python
from lmstudio import LMStudio

client = LMStudio()
model = client.llm.load("lmstudio-community/Meta-Llama-3.1-8B-Instruct-GGUF")
response = model.respond([
    {"role": "user", "content": "Hello!"}
])
```
[1]

### CLI (lms)

```bash
lms load llama-3.1-8b       # Load a model
lms unload                    # Unload current model
lms ls                        # List available models
lms server start              # Start the local server
lms server stop               # Stop the local server
```
[1]

### Offline Document RAG

Attach documents to conversations for offline retrieval-augmented generation. LM Studio processes documents locally without sending data to external services. [1]

## Use Cases

### Local Development and Testing

Run LLMs locally for development without API costs. OpenAI and Anthropic-compatible endpoints enable testing with the same client code used for cloud APIs. [1]

### Privacy-First Applications

Process sensitive data entirely on-device. All inference happens locally with no data leaving the machine. [1]

### Coding Assistant Integration

LM Studio integrates with coding tools via its OpenAI-compatible endpoint and MCP server support. [1]

### Prototyping

Use the built-in chat interface to quickly test different models and prompts before committing to a production setup. [1]

## API Reference Summary

### OpenAI-Compatible Endpoints

- `POST /v1/chat/completions` -- Chat completions
- `POST /v1/completions` -- Text completions
- `POST /v1/embeddings` -- Vector embeddings
- `GET /v1/models` -- List loaded models

### Anthropic-Compatible Endpoints

- `POST /v1/messages` -- Anthropic Messages API

### Native API

- `POST /api/v1/chat/completions` -- Native chat endpoint
- `GET /api/v1/models` -- Model listing with extended metadata

### SDK Methods

- `client.llm.load(model)` -- Load a model
- `model.respond(messages)` -- Generate response
- `model.stream(messages)` -- Stream response [1]

## Configuration and Customization

### Generation Parameters

- **Temperature** -- Randomness control (0.0-2.0)
- **Top P** -- Nucleus sampling threshold
- **Top K** -- Top-k sampling limit
- **Max Tokens** -- Maximum output tokens
- **Repeat Penalty** -- Repetition control
- **Context Length** -- Model context window size

### Server Configuration

- **Port** -- Default: 1234
- **CORS** -- Cross-origin request settings
- **GPU Layers** -- Number of model layers offloaded to GPU
- **Threads** -- CPU thread count for inference

### Per-Model Defaults

Configure default parameters for each model, including system prompts, temperature, and context length. [1]

## Integration Patterns

### With OpenAI SDK

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")
```

### With Anthropic SDK

```python
import anthropic
client = anthropic.Anthropic(base_url="http://localhost:1234/v1", api_key="lm-studio")
```

### With LangChain

Use LM Studio as an OpenAI-compatible backend for LangChain chains and agents.

### With MCP Servers

Connect external tools and data sources via the Model Context Protocol.

### With IDE Extensions

VS Code, JetBrains, and other IDE extensions connect via the OpenAI-compatible API.

## Examples

### OpenAI Chat Completion

```bash
curl http://localhost:1234/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "lmstudio-community/Meta-Llama-3.1-8B-Instruct-GGUF",
    "messages": [
      {"role": "system", "content": "You are a pirate."},
      {"role": "user", "content": "Introduce yourself."}
    ],
    "temperature": 0.7,
    "max_tokens": 256
  }'
```
[1]

### Generate Embeddings

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:1234/v1", api_key="lm-studio")
response = client.embeddings.create(
    model="nomic-embed-text-v1.5",
    input="The quick brown fox jumps over the lazy dog",
)
print(response.data[0].embedding[:5])
```
[1]

## Limitations and Considerations

- **Proprietary** -- Not open source; source code not available for modification
- **Hardware requirements** -- 16GB+ RAM recommended; larger models need more memory or GPU VRAM
- **Model format** -- Only supports GGUF and MLX formats; no native PyTorch or ONNX support
- **GPU support** -- NVIDIA CUDA, Apple Metal supported; AMD ROCm support limited
- **Performance** -- Local inference is slower than cloud GPU-powered APIs for large models
- **Desktop dependency** -- Full app requires desktop environment; llmster provides headless mode
- **Model availability** -- Limited to models available in GGUF/MLX format on HuggingFace [1]

## Changelog Highlights

- **v0.4.1** -- Anthropic-compatible endpoint (POST /v1/messages)
- **MCP integration** -- Model Context Protocol server support
- **Speculative decoding** -- Performance optimization for faster generation
- **Continuous batching** -- Parallel request handling
- **llmster** -- Headless daemon for server deployments
- **TypeScript SDK** -- Native TypeScript client library
- **Python SDK** -- Native Python client library
- **MLX support** -- Apple Silicon-optimized model format
- **Document RAG** -- Offline document attachment and querying [1]

## Citations

- [1] LM Studio Documentation - <https://lmstudio.ai/docs>
