# Ollama

> Local LLM runner with REST API and CLI for quantized models

| Field | Value |
|-------|-------|
| Group | Local Model Runtimes |
| Type | Infra/API/UI |
| Open Source | Yes |
| GitHub | [https://github.com/ollama/ollama](https://github.com/ollama/ollama) |
| Stars | 171390 |
| Documentation | [Official Docs](https://docs.ollama.com/) |

## Overview

Ollama is an open-source framework for running large language models locally. It packages model weights, configuration, and runtime into a single streamlined interface, enabling developers to download, run, and manage LLMs with simple CLI commands or a REST API. Ollama uses llama.cpp as its inference backend, supporting quantized GGUF model formats for efficient execution on consumer hardware. [1]

The platform provides an extensive model library at ollama.com/library with pre-packaged models including Llama, Qwen, DeepSeek, Gemma, Mistral, and hundreds of others. Models can be customized via Modelfiles, and Ollama exposes an OpenAI-compatible REST API on localhost, making it a drop-in local replacement for cloud LLM APIs. Official SDKs are available for Python and JavaScript. [1][2]

## Core Concepts

### Model Library

Ollama maintains a curated model library at ollama.com/library where models are tagged with versions and quantization levels (e.g., `llama3.1:8b-q4_0`). Models are downloaded on first use and cached locally. The library includes language models, vision models, and embedding models. [1]

### Modelfile

A Modelfile is a configuration file for creating and customizing models in Ollama, similar to a Dockerfile. It specifies the base model, system prompt, parameters (temperature, context length, etc.), template format, and adapter weights (LoRA). [1]

### llama.cpp Backend

Ollama uses the llama.cpp project (founded by Georgi Gerganov) as its primary inference engine. This C++ library provides efficient CPU and GPU inference for quantized models in GGUF format, enabling LLM execution on consumer hardware without requiring high-end GPUs. [1]

### Quantization

Models in Ollama's library are pre-quantized to various levels (Q4_0, Q4_K_M, Q5_K_M, Q8_0, FP16) to trade off between model quality and memory/speed requirements. The `/api/create` endpoint can also quantize models on the fly. [1][2]

## Architecture

### System Architecture

- **CLI** -- Command-line interface for model management and interactive chat
- **REST API Server** -- HTTP server on port 11434 providing all API endpoints
- **Model Manager** -- Handles model downloads, caching, and lifecycle management
- **llama.cpp Runtime** -- Core inference engine executing quantized models
- **GPU Backend** -- Optional GPU acceleration via CUDA (NVIDIA), ROCm (AMD), or Metal (Apple)

### Model Storage

Models are stored in `~/.ollama/models/` (default) with manifest files describing layers, quantization, and configuration. The blob storage system enables efficient sharing of common layers between model variants. [1]

### Request Flow

Client requests hit the REST API, which routes to the appropriate model instance. Models are loaded into memory on first request and kept warm for a configurable duration (`keep_alive`). Multiple concurrent requests are supported. [2]

## Key Features and Functionality

### CLI Commands

```bash
ollama run llama3.1          # Download and run a model
ollama pull gemma3            # Download a model without running
ollama list                   # List installed models
ollama show llama3.1          # Show model details
ollama create mymodel -f Modelfile  # Create custom model
ollama rm llama3.1            # Remove a model
ollama cp llama3.1 mymodel    # Copy a model
ollama launch                 # Launch integrations
```
[1]

### REST API

The API exposes endpoints on `http://localhost:11434`:

- `POST /api/generate` -- Text generation with streaming
- `POST /api/chat` -- Chat completion with message history
- `POST /api/create` -- Create custom models
- `GET /api/tags` -- List local models
- `POST /api/show` -- Show model information
- `POST /api/copy` -- Copy a model
- `DELETE /api/delete` -- Delete a model
- `POST /api/pull` -- Download a model
- `POST /api/push` -- Upload a model
- `POST /api/embeddings` -- Generate embeddings
- `GET /api/ps` -- List running models
- `GET /api/version` -- API version [2]

### Structured Outputs

Pass a JSON schema via the `format` parameter to enforce structured responses:

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1",
  "messages": [{"role": "user", "content": "List 3 colors"}],
  "format": {
    "type": "object",
    "properties": {
      "colors": {"type": "array", "items": {"type": "string"}}
    },
    "required": ["colors"]
  },
  "stream": false
}'
```
[2]

### Tool Calling

The chat API supports tool/function calling where the model can request external function execution:

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1",
  "messages": [{"role": "user", "content": "What is the weather in NYC?"}],
  "tools": [{
    "type": "function",
    "function": {
      "name": "get_weather",
      "description": "Get weather for a location",
      "parameters": {
        "type": "object",
        "properties": {"location": {"type": "string"}},
        "required": ["location"]
      }
    }
  }]
}'
```
[2]

### Multimodal Vision

Vision-capable models accept base64-encoded images:

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llava",
  "messages": [{
    "role": "user",
    "content": "What is in this image?",
    "images": ["<base64-encoded-image>"]
  }]
}'
```
[2]

## Use Cases

### Local Development

Run LLMs locally for development and testing without API costs or internet dependency. The OpenAI-compatible API enables testing with the same client code used for cloud APIs. [1]

### Privacy-Sensitive Applications

Process data entirely on-premises without sending information to external servers. Suitable for healthcare, legal, and financial applications with data residency requirements. [1]

### Coding Assistants

Ollama integrates with coding tools including Claude Code, Codex, VS Code extensions, and other IDE integrations via the `ollama launch` command. [1]

### Edge Deployment

Deploy LLMs on edge devices or air-gapped networks where cloud API access is unavailable. Quantized models run efficiently on consumer CPUs and GPUs. [1]

## API Reference Summary

### Generate API

```python
import ollama

response = ollama.generate(
    model="llama3.1",
    prompt="Why is the sky blue?",
)
print(response["response"])
```

### Chat API

```python
import ollama

response = ollama.chat(
    model="llama3.1",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"},
    ],
)
print(response["message"]["content"])
```

### Embeddings API

```python
import ollama

response = ollama.embed(
    model="llama3.1",
    input="The quick brown fox",
)
print(response["embeddings"])
```
[1]

## Configuration and Customization

### Environment Variables

- **`OLLAMA_HOST`** -- Bind address (default: `127.0.0.1:11434`)
- **`OLLAMA_MODELS`** -- Model storage directory
- **`OLLAMA_KEEP_ALIVE`** -- Duration to keep models loaded (default: `5m`)
- **`OLLAMA_NUM_PARALLEL`** -- Max parallel requests per model
- **`OLLAMA_MAX_LOADED_MODELS`** -- Max models loaded simultaneously
- **`OLLAMA_GPU_OVERHEAD`** -- Reserved GPU memory (bytes)
- **`OLLAMA_FLASH_ATTENTION`** -- Enable flash attention (`1` to enable)

### Modelfile Parameters

```
FROM llama3.1
PARAMETER temperature 0.7
PARAMETER num_ctx 4096
PARAMETER top_p 0.9
PARAMETER top_k 40
PARAMETER repeat_penalty 1.1
SYSTEM "You are a helpful coding assistant."
```

### API Parameters

- **`temperature`** -- Randomness control (0.0-2.0)
- **`num_ctx`** -- Context window size
- **`top_p`** / **`top_k`** -- Sampling parameters
- **`keep_alive`** -- How long to keep model loaded after request
- **`stream`** -- Enable/disable streaming (default: true)
- **`raw`** -- Bypass prompt templating [1][2]

## Integration Patterns

### With OpenAI SDK

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")
response = client.chat.completions.create(
    model="llama3.1",
    messages=[{"role": "user", "content": "Hello!"}],
)
```

### With LangChain

Ollama provides a LangChain integration as an LLM backend for chains, agents, and RAG pipelines.

### With Web UIs (Open WebUI, LibreChat)

Multiple open-source web interfaces connect to Ollama's API for browser-based chat experiences.

### With IDE Extensions

VS Code, JetBrains, Neovim, and Emacs extensions use Ollama for local code completion and chat.

### With RAG Frameworks

Ollama's embedding API enables local vector search for retrieval-augmented generation without external API calls.

## Examples

### Chat Completion

```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1",
  "messages": [
    {"role": "system", "content": "You are a pirate."},
    {"role": "user", "content": "Tell me about yourself."}
  ],
  "stream": false
}'
```
[2]

### Python SDK

```python
import ollama

# Streaming chat
for chunk in ollama.chat(
    model="llama3.1",
    messages=[{"role": "user", "content": "Write a haiku about AI"}],
    stream=True,
):
    print(chunk["message"]["content"], end="")
```
[1]

### Custom Model Creation

```bash
# Create a Modelfile
cat <<EOF > Modelfile
FROM llama3.1
SYSTEM "You are Mario from Super Mario Bros."
PARAMETER temperature 0.8
EOF

# Create and run the custom model
ollama create mario -f Modelfile
ollama run mario
```
[1]

## Limitations and Considerations

- **Model quality vs size** -- Quantized models trade accuracy for efficiency; larger quantizations (Q8, FP16) are more accurate but require more memory
- **Hardware requirements** -- Larger models (70B+) require significant RAM or VRAM; 7B models run well on 8GB+ systems
- **GPU support** -- Requires NVIDIA (CUDA), AMD (ROCm), or Apple (Metal) for GPU acceleration; falls back to CPU
- **Concurrent requests** -- Performance degrades with many concurrent requests on limited hardware
- **Model availability** -- Not all models are available in Ollama's library; custom GGUF imports are supported
- **Context window** -- Default context is model-dependent; can be adjusted but limited by available memory [1]

## Changelog Highlights

- **Ollama launch** -- Integration with coding assistants (Claude Code, Codex)
- **Tool calling** -- Function calling support in chat API
- **Structured outputs** -- JSON schema enforcement on responses
- **Vision support** -- Multimodal models with image inputs
- **Embedding API** -- Vector embedding generation
- **OpenAI compatibility** -- Drop-in API compatibility for OpenAI clients
- **Docker support** -- Official Docker images with GPU passthrough
- **Python and JavaScript SDKs** -- Official client libraries [1][2]

## Citations

- [1] Ollama GitHub Repository - <https://github.com/ollama/ollama>
- [2] Ollama API Reference - <https://github.com/ollama/ollama/blob/main/docs/api.md>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- ollama
- local llm
- local model runtime
- llama.cpp
- gguf
- quantization
- modelfile
- on-device inference
- offline llm
- llama3
- gemma
- qwen
- deepseek
- mistral
- llava
- ollama rest api
- localhost:11434
- ollama cli
- openai-compatible local
- ollama python sdk
- ollama javascript sdk
- ollama embeddings
- ollama tool calling
- ollama vision
- model library
- keep_alive
- cuda rocm metal
- air-gapped llm

### Verb-Noun Tasks

- Run a quantized LLM locally from a CLI
- Pull a model from the Ollama library
- Expose an OpenAI-compatible endpoint on localhost
- Generate embeddings locally without leaving the machine
- Customize a model with a Modelfile and SYSTEM prompt
- Enforce a JSON schema on a local model response
- Call a tool/function from a locally hosted model
- Send a base64 image to a vision-capable local model
- Stream tokens from `/api/chat` via curl
- Keep a model warm in memory with `keep_alive`
- Run multiple parallel requests against a single local model
- Integrate a local model into LangChain, VS Code, or Claude Code
- Import a custom GGUF file as a private Ollama model
- Quantize a model on the fly via `/api/create`

### User Intent Phrases

- How do I run Llama 3 on my laptop without paying for an API?
- I want an OpenAI-compatible API that runs entirely on my machine.
- Can I do RAG without sending data to the cloud?
- What is the easiest way to try DeepSeek or Qwen locally?
- I need a coding assistant that works offline.
- Show me how to call a local LLM from Python using the OpenAI SDK.
- How do I add tool calling to a local model?
- What is a Modelfile and how is it different from a Dockerfile?
- How do I serve a vision model on my own GPU?
- Which quantization should I pick for 8GB of VRAM?
- How do I keep a local model loaded between requests?
- Can I run an LLM on an air-gapped machine?
- I want to test prompts locally before paying for OpenAI.
- How do I share Ollama across my LAN?

### Problem Statements

- API costs are growing and I want to move inference on-device.
- Data residency rules forbid sending user prompts to a cloud provider.
- I have a workstation GPU sitting idle that could run my model.
- I need to prototype with several models without paying per token.
- Cloud LLM latency is too variable for my desktop tool.
- My environment is air-gapped and cloud APIs are not reachable.
- I want to test agent loops with no risk of runaway provider bills.

### When to Pick This

- Pick this when you want a CLI-first, scriptable local runtime over a desktop GUI (vs LM Studio).
- Pick this when you need a long-running REST API as a daemon, not a desktop app session.
- Pick this when you want a public model library you can `pull` by name and tag.
- Pick this when you need to ship Modelfiles in source control to standardize local model configuration.
- Pick this when you want an open-source local runtime you can fork, audit, and embed in CI.
- Pick this when you need Docker images with GPU passthrough for a homelab deployment.
- Pick this when you prefer llama.cpp/GGUF over Apple's MLX format.

### Related Terms and Aliases

- llama.cpp wrapper
- local inference server
- on-prem LLM
- private LLM
- offline AI
- self-hosted llm
- GGUF runner
- localhost LLM
- consumer-GPU inference
- edge LLM
