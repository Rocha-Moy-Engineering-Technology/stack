# Max

> High-performance inference framework with cross-vendor hardware support

| Field | Value |
|-------|-------|
| Group | Inference Serving |
| Type | SDK/Infra |
| Open Source | Yes |
| GitHub | [https://github.com/modular/modular](https://github.com/modular/modular) |
| Stars | 25622 |
| Documentation | [Official Docs](https://docs.modular.com/max/) |

## Overview

MAX is Modular's AI inference and deployment platform designed to accelerate AI inference and abstract hardware complexity. The platform enables deployment of generative AI models through Docker containers with minimal setup, providing an OpenAI-compatible endpoint that supports 500+ optimized models from HuggingFace. Every model is optimized using MAX Graph to ensure performance and portability across architectures. [1][2]

MAX offers three primary workflows: serving (Docker-based deployment with OpenAI-compatible endpoints), deployment via Mammoth (cloud-scale deployment across diverse GPU infrastructure), and development (custom operator writing with Mojo, a Python-style language for CPU and GPU programming). The latest stable release is version 26.1. [1]

## Core Concepts

### MAX Graph

MAX Graph is the internal compilation and optimization framework that transforms model definitions into hardware-optimized execution graphs. It ensures every model runs with optimal performance regardless of the target hardware, providing automatic kernel selection and fusion optimizations. [1]

### Mojo Language

Mojo is Modular's Python-style programming language that enables developers to write code for both CPUs and GPUs. It provides low-level hardware control while maintaining Python-like syntax, enabling custom operator development and hardware-agnostic GPU kernel writing. [1]

### OpenAI-Compatible Endpoint

MAX exposes an OpenAI-compatible API at `/v1/`, allowing any OpenAI SDK client to connect by changing the base URL. This minimizes migration friction when moving from cloud APIs to self-hosted inference. [2]

### Hardware Abstraction

MAX handles deployment across heterogeneous GPU clusters, abstracting away hardware-specific complexities. Supported hardware includes NVIDIA (B200, H200, H100) and AMD (MI355X, MI325X, MI300X) GPUs. [1][2]

## Architecture

### Platform Components

- **MAX Serve** -- Model serving endpoint with OpenAI-compatible API
- **MAX Graph** -- Compilation and optimization framework for model graphs
- **MAX Benchmark** -- Performance evaluation and comparison tool
- **Mammoth** -- Cloud-scale deployment engine for heterogeneous GPU clusters
- **Mojo Runtime** -- Language runtime for custom operators and GPU kernels

### Serving Architecture

MAX Serve loads a model, compiles it through MAX Graph with hardware-specific optimizations, and exposes it via an HTTP endpoint. The server handles request batching, memory management, and model lifecycle automatically. [1][2]

### Model Pipeline

1. Model weights loaded from HuggingFace or local path
2. MAX Graph compiles the model with hardware-specific optimizations
3. Optimized model deployed to serving endpoint
4. Requests processed through continuous batching with paged attention

## Key Features and Functionality

### Model Serving

```bash
export HF_TOKEN="hf_..."
max serve --model google/gemma-3-27b-it
```

For models requiring code execution:

```bash
max serve --model google/gemma-3-27b-it --trust-remote-code
```
[2]

### OpenAI-Compatible API

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")

completion = client.chat.completions.create(
    model="google/gemma-3-27b-it",
    messages=[{"role": "user", "content": "Who won the world series in 2020?"}],
)
print(completion.choices[0].message.content)
```
[2]

### Multimodal Support

Process images alongside text for vision-language models:

```python
completion = client.chat.completions.create(
    model="google/gemma-3-27b-it",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Write a caption for this image"},
            {"type": "image_url", "image_url": {"url": "https://example.com/image.jpg"}}
        ]
    }],
    max_tokens=300,
)
```
[2]

### Benchmarking

Text-focused performance evaluation:

```bash
max benchmark \
  --model google/gemma-3-27b-it \
  --backend modular \
  --endpoint /v1/chat/completions \
  --dataset-name sonnet \
  --num-prompts 500 \
  --sonnet-input-len 550 \
  --output-lengths 256
```

Save results to JSON:

```bash
max benchmark ... --save-result --result-filename "results.json"
```
[2]

### Custom Operators

MAX provides full extensibility for writing custom ops, hardware-agnostic GPU kernels, and specialized model optimizations using Mojo. [1]

## Use Cases

### Self-Hosted LLM Serving

Deploy open-source LLMs with production-grade performance behind an OpenAI-compatible API, enabling private hosting with minimal code changes from cloud APIs. [1][2]

### Multi-Hardware Deployment

Deploy models across mixed GPU fleets (NVIDIA and AMD) with a single codebase, using MAX's hardware abstraction to optimize for each target. [1]

### Model Benchmarking

Compare inference performance across models, backends, and hardware configurations using the built-in benchmark tool. [2]

### Custom Model Optimization

Write custom operators and GPU kernels in Mojo for specialized model architectures or domain-specific optimizations. [1]

## API Reference Summary

### CLI Commands

- `max serve --model <model>` -- Start serving endpoint
- `max benchmark --model <model>` -- Run benchmarks
- `max --help` -- Show available commands

### Server Endpoints

- `POST /v1/chat/completions` -- Chat completions
- `POST /v1/completions` -- Text completions
- `POST /v1/embeddings` -- Vector embeddings
- `GET /health` -- Health check
- `GET /v1/models` -- List loaded models [2]

## Configuration and Customization

### Serve Parameters

- **`--model`** -- HuggingFace model ID or local path
- **`--device-memory-utilization`** -- Fraction of GPU memory to use (default: 0.9)
- **`--trust-remote-code`** -- Allow models with custom code
- **`--host`** -- Bind address (default: 0.0.0.0)
- **`--port`** -- Server port (default: 8000)

### Benchmark Parameters

- **`--backend`** -- Inference backend (modular, vllm, etc.)
- **`--endpoint`** -- API endpoint to benchmark
- **`--dataset-name`** -- Dataset for benchmarking (sonnet, random, etc.)
- **`--num-prompts`** -- Number of test prompts
- **`--output-lengths`** -- Expected output token lengths
- **`--save-result`** -- Save results to file [2]

### Environment Variables

- **`HF_TOKEN`** -- HuggingFace access token for gated models
- **`MAX_LOG_LEVEL`** -- Logging verbosity [2]

## Integration Patterns

### With OpenAI SDK

MAX's OpenAI-compatible API enables direct connection from any OpenAI client library:

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")
```

### With Agent Frameworks (LangChain, LangGraph)

Integrates as an LLM backend through the OpenAI-compatible endpoint.

### With Docker/Kubernetes

Deploy via Docker containers for production environments with GPU passthrough.

### With vLLM Comparison

MAX benchmark tool can compare performance against vLLM and other backends.

## Examples

### Basic Chat

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="EMPTY")
response = client.chat.completions.create(
    model="google/gemma-3-27b-it",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum computing."},
    ],
    temperature=0.7,
)
print(response.choices[0].message.content)
```
[2]

### Multimodal Image Analysis

```python
response = client.chat.completions.create(
    model="google/gemma-3-27b-it",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "Describe this image"},
            {"type": "image_url", "image_url": {"url": "https://example.com/photo.jpg"}}
        ]
    }],
    max_tokens=300,
)
```
[2]

## Limitations and Considerations

- **Linux/WSL only** -- Currently requires Linux or Windows Subsystem for Linux
- **GPU required** -- Production performance requires NVIDIA or AMD GPU; CPU inference not emphasized
- **Initial compilation** -- First model load includes compilation time (several minutes for large models)
- **Memory management** -- Out-of-memory issues addressed by reducing `--device-memory-utilization` below default 0.9
- **Model compatibility** -- Supports 500+ models but not all HuggingFace architectures
- **Mojo maturity** -- Mojo language ecosystem is still developing for custom operator creation [1][2]

## Changelog Highlights

- **MAX 26.1** -- Latest stable release (January 2026)
- **OpenAI-compatible API** -- Drop-in API compatibility for serving
- **500+ model support** -- Broad HuggingFace model compatibility
- **Mammoth deployment** -- Cloud-scale heterogeneous GPU deployment
- **MAX Benchmark** -- Built-in performance evaluation tool
- **Mojo integration** -- Custom operator development in Python-style language
- **Multi-hardware support** -- NVIDIA and AMD GPU optimization
- **MAX Graph** -- Automatic model compilation and optimization [1][2]

## Citations

- [1] MAX Platform Documentation - <https://docs.modular.com/max/>
- [2] MAX Get Started Guide - <https://docs.modular.com/max/get-started>
