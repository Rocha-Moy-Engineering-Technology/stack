# vLLM

> High-throughput memory-efficient LLM inference and serving engine

| Field | Value |
|-------|-------|
| Group | Inference Engines |
| Type | SDK/Infra |
| Open Source | Yes |
| GitHub | [https://github.com/vllm-project/vllm](https://github.com/vllm-project/vllm) |
| Stars | 80008 |
| Documentation | [Official Docs](https://docs.vllm.ai/) |

## Overview

vLLM is an open-source large language model inference and serving engine designed for high-throughput, low-latency deployment. Developed at UC Berkeley's Sky Computing Lab, it provides both offline batched inference and an OpenAI-compatible online serving API. The library's core innovation is PagedAttention, an attention algorithm that efficiently manages key-value (KV) cache memory, enabling significantly higher throughput compared to naive implementations. [1][2]

vLLM supports most popular open-source models on HuggingFace including transformer-based LLMs (Llama, Qwen, Mistral), mixture-of-expert models (Mixtral, DeepSeek-V2/V3), embedding models (E5-Mistral), and multi-modal LLMs (LLaVA). Hardware support spans NVIDIA GPUs, AMD GPUs, Intel XPU, PowerPC, Arm CPUs, TPUs, Intel Gaudi, IBM Spyre, and Huawei Ascend. [2]

## Core Concepts

### PagedAttention

PagedAttention is vLLM's fundamental memory management innovation. It organizes the KV cache into fixed-size pages (blocks), similar to virtual memory in operating systems. This eliminates memory fragmentation during inference and enables near-optimal memory utilization, allowing more concurrent requests to be served from the same GPU memory. [1]

### Continuous Batching

Rather than waiting for an entire batch of requests to complete before processing new ones, vLLM dynamically inserts new requests into the processing pipeline as slots become available. This continuous batching approach significantly improves GPU utilization and overall throughput compared to static batching. [1]

### Speculative Decoding

vLLM accelerates generation through draft models that predict multiple tokens ahead, which are then verified by the main model in a single forward pass. This reduces the number of sequential forward passes needed, lowering latency for compatible workloads without changing output quality. [1]

### Prefix Caching

Prefix caching allows vLLM to reuse KV cache computations from previous requests that share common prefixes (e.g., system prompts). This avoids redundant computation for repeated context, reducing latency and compute cost for applications with shared prompt templates. [1]

## Architecture

### Engine Architecture

vLLM's architecture consists of several key components:

- **LLM Engine** -- Core inference engine that manages model execution, KV cache allocation via PagedAttention, and request scheduling
- **AsyncLLMEngine** -- Asynchronous wrapper for the engine enabling non-blocking request handling in server mode
- **Scheduler** -- Determines which requests to process in each iteration using continuous batching
- **Block Manager** -- Manages physical GPU and CPU memory blocks for the KV cache
- **Model Runner** -- Executes the actual model forward passes with optimized kernels

### Serving Layer

The OpenAI-compatible API server wraps the async engine, providing REST endpoints for chat completions, completions, and embeddings that are drop-in compatible with OpenAI client libraries. [1]

### Distributed Execution

vLLM supports multiple parallelism strategies for large models:

- **Tensor parallelism** -- Splits model layers across GPUs
- **Pipeline parallelism** -- Distributes model stages across GPUs
- **Expert parallelism** -- Distributes MoE experts across GPUs
- **Data parallelism** -- Replicates the model for higher throughput [2]

## Key Features and Functionality

### OpenAI-Compatible Server

```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --port 8000
```

The server exposes `/v1/chat/completions`, `/v1/completions`, and `/v1/embeddings` endpoints compatible with OpenAI client libraries. [1]

### Offline Batched Inference

```python
from vllm import LLM, SamplingParams

llm = LLM(model="meta-llama/Llama-3.1-8B-Instruct")
sampling_params = SamplingParams(temperature=0.8, top_p=0.95)

prompts = ["Hello, my name is", "The capital of France is"]
outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    print(output.outputs[0].text)
```
[1]

### Quantization

Comprehensive quantization support for reduced memory and faster inference:

- **GPTQ** -- Post-training quantization with calibration data
- **AWQ** -- Activation-aware weight quantization
- **FP8** -- 8-bit floating point (W8A8)
- **INT8** -- 8-bit integer (W8A8)
- **INT4** -- 4-bit integer (W4A16)
- **BitsAndBytes** -- Dynamic quantization
- **GGUF** -- llama.cpp-compatible quantized formats
- **AutoRound** -- Intel automated quantization
- **TorchAO** -- PyTorch-native quantization [1][2]

### Multi-LoRA Serving

Serve multiple LoRA adapters simultaneously on the same base model, dynamically routing requests to the appropriate adapter. This enables efficient multi-tenant deployments with fine-tuned variants. [2]

### Multimodal Support

vLLM handles vision-language models (VLMs) that process both text and images, enabling use cases like image captioning, visual question answering, and document analysis with models like LLaVA. [1]

### Structured Outputs

Generate outputs conforming to JSON schemas, regular expressions, or context-free grammars using guided decoding. [1]

## Use Cases

### High-Throughput API Serving

Deploy LLMs as production API endpoints with OpenAI-compatible interfaces. Continuous batching and PagedAttention maximize throughput per GPU, making it cost-effective for high-traffic applications. [1]

### Batch Processing

Process large datasets offline with the `LLM` class for batch inference tasks like text classification, summarization, or data extraction where latency is less critical than throughput. [1]

### Multi-Model Serving

Use LoRA adapter support to serve multiple fine-tuned model variants from a single base model deployment, reducing GPU memory requirements for multi-tenant applications. [2]

### Distributed Large Model Inference

Deploy models too large for a single GPU across multiple GPUs or nodes using tensor and pipeline parallelism, enabling inference on frontier-scale models. [2]

## API Reference Summary

### Python API

- `LLM(model, ...)` -- Create offline inference engine
- `LLM.generate(prompts, sampling_params)` -- Generate completions
- `LLM.chat(messages, ...)` -- Chat-style generation
- `LLM.encode(inputs)` -- Generate embeddings
- `SamplingParams(temperature, top_p, max_tokens, ...)` -- Control generation

### CLI

- `vllm serve <model>` -- Start OpenAI-compatible server
- `vllm complete <model> <prompt>` -- Quick offline completion
- `vllm chat <model>` -- Interactive chat session

### Server Endpoints

- `POST /v1/chat/completions` -- Chat completions (OpenAI-compatible)
- `POST /v1/completions` -- Text completions
- `POST /v1/embeddings` -- Vector embeddings
- `GET /health` -- Health check
- `GET /v1/models` -- List loaded models [1]

## Configuration and Customization

### Engine Arguments

- **`--model`** -- HuggingFace model ID or local path
- **`--tensor-parallel-size`** -- Number of GPUs for tensor parallelism
- **`--pipeline-parallel-size`** -- Number of pipeline stages
- **`--gpu-memory-utilization`** -- Fraction of GPU memory to use (default: 0.9)
- **`--max-model-len`** -- Maximum sequence length
- **`--dtype`** -- Model data type (auto, float16, bfloat16, float32)
- **`--quantization`** -- Quantization method (awq, gptq, fp8, etc.)
- **`--enable-prefix-caching`** -- Enable automatic prefix caching
- **`--max-num-seqs`** -- Maximum concurrent sequences

### Sampling Parameters

- **`temperature`** -- Randomness (0.0 = greedy)
- **`top_p`** -- Nucleus sampling threshold
- **`top_k`** -- Top-k sampling
- **`max_tokens`** -- Maximum output tokens
- **`presence_penalty`** / **`frequency_penalty`** -- Repetition control
- **`stop`** -- Stop sequences
- **`n`** -- Number of completions per prompt [1]

## Integration Patterns

### With LLM Providers (OpenAI SDK)

vLLM's OpenAI-compatible API allows any OpenAI SDK client to connect by changing the base URL:

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:8000/v1", api_key="token")
```

### With Agent Frameworks (LangChain, LangGraph)

Integrates as an LLM backend through the OpenAI-compatible endpoint or direct Python API.

### With Deployment Platforms (Ray Serve, Modal, BentoML, SkyPilot)

vLLM provides deployment guides for Ray Serve (distributed serving), Modal (serverless GPU), BentoML (model packaging), and SkyPilot (multi-cloud). [1]

### With Kubernetes

Deploy using Helm charts or custom manifests with NVIDIA GPU operator for production Kubernetes clusters.

### With Observability (Prometheus)

Exposes Prometheus-compatible metrics for monitoring request latency, throughput, queue depth, and GPU utilization.

## Examples

### OpenAI-Compatible Chat

```python
from openai import OpenAI

client = OpenAI(base_url="http://localhost:8000/v1", api_key="unused")

response = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain quantum computing in simple terms."}
    ],
    temperature=0.7,
    max_tokens=256,
)
print(response.choices[0].message.content)
```
[1]

### Offline Batch Inference with Streaming

```python
from vllm import LLM, SamplingParams

llm = LLM(model="meta-llama/Llama-3.1-8B-Instruct")

for output in llm.generate(
    ["Write a poem about AI"],
    SamplingParams(temperature=0.9, max_tokens=200),
):
    print(output.outputs[0].text)
```
[1]

## Limitations and Considerations

- **GPU memory** -- Large models require significant GPU memory; quantization can reduce requirements but may affect quality
- **Model compatibility** -- Not all HuggingFace models are supported; check the supported models list
- **Cold start** -- Initial model loading takes time, especially for large models; prefix caching helps for repeated prompts
- **CPU inference** -- CPU support is available but significantly slower than GPU inference
- **Speculative decoding** -- Requires compatible draft model selection; not all model pairs work well together
- **Memory fragmentation** -- While PagedAttention reduces fragmentation, very long sequences can still cause out-of-memory issues [1][2]

## Changelog Highlights

- **PagedAttention** -- Core memory management innovation enabling efficient KV cache
- **Continuous batching** -- Dynamic request scheduling for improved throughput
- **Multi-LoRA** -- Simultaneous serving of multiple LoRA adapters
- **Speculative decoding** -- Draft-model acceleration for lower latency
- **Multimodal support** -- Vision-language model inference
- **Structured outputs** -- JSON schema and grammar-guided generation
- **FP8 quantization** -- Hardware-accelerated 8-bit inference
- **Expert parallelism** -- Efficient MoE model distribution
- **CUDA/HIP graph** -- Kernel launch overhead reduction [1][2]

## Citations

- [1] vLLM Documentation - <https://docs.vllm.ai/>
- [2] vLLM GitHub Repository - <https://github.com/vllm-project/vllm>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

vLLM, PagedAttention, continuous batching, KV cache, prefix caching, speculative decoding, OpenAI-compatible server, tensor parallelism, pipeline parallelism, expert parallelism, multi-LoRA, AWQ, GPTQ, FP8 quantization, INT4, BitsAndBytes, GGUF, AsyncLLMEngine, SamplingParams, structured outputs, guided decoding, LLM serving, high-throughput inference, GPU inference, MoE, Mixtral, Llama, Qwen, LLaVA

### Verb-Noun Tasks

- Serve a Hugging Face model as an OpenAI-compatible API with `vllm serve`
- Run offline batched inference via `LLM.generate(prompts, SamplingParams)`
- Enable PagedAttention for higher concurrent request throughput
- Turn on `--enable-prefix-caching` to reuse shared system-prompt KV cache
- Distribute a 70B model across GPUs with `--tensor-parallel-size`
- Serve multiple LoRA adapters off one base model
- Quantize a model with AWQ, GPTQ, FP8, or INT4 for less VRAM
- Run vision-language inference with LLaVA
- Generate guided JSON, regex, or CFG-conformant output
- Expose Prometheus metrics for latency, throughput, and GPU utilization
- Deploy vLLM on Kubernetes, Ray Serve, Modal, or BentoML
- Accelerate decoding with a draft model via speculative decoding

### User Intent Phrases

- "How do I self-host Llama 3.1 with an OpenAI-compatible API?"
- "I want maximum tokens-per-second on my A100/H100"
- "How do I serve multiple LoRA fine-tunes from one base model?"
- "What's the fastest open-source inference engine for production?"
- "How do I run 70B+ models split across multiple GPUs?"
- "I need to reduce VRAM with quantization (AWQ/GPTQ/FP8)"
- "How do I cache system prompts to cut latency?"
- "How do I get OpenAI SDK compatibility without paying OpenAI?"
- "I want to run a multi-modal vision-language model locally"
- "How do I batch-process millions of prompts offline?"

### Problem Statements

- Naive HuggingFace `generate()` wastes GPU memory and underutilizes batching
- KV cache fragmentation limits concurrent requests per GPU
- Repeated system prompts repay full prefill cost every request
- Large models exceed single-GPU memory; sharding is non-trivial
- Multi-tenant fine-tuned models require N copies of the base weights
- Closed APIs cost too much at scale or can't run private/regulated data on-prem
- JSON-mode output from open models is unreliable without constrained decoding

### When to Pick This

- Pick this when throughput-per-GPU is the dominant cost driver
- Pick this over SGLang when broad hardware support (NVIDIA + AMD + Intel + TPU + Gaudi + Spyre + Ascend) matters
- Pick this over Triton Inference Server when the workload is LLM-only and OpenAI-compat is desired
- Pick this when multi-LoRA serving on one base model is required
- Pick this when you need the broadest open-weight model coverage on HuggingFace
- Pick this over Hugging Face Transformers raw generate() for any production-scale workload
- Pick this over hosted APIs when data residency, custom models, or quantization control matter

### Related Terms and Aliases

- vLLM project, Sky Computing Lab vLLM
- "OpenAI-compatible local server"
- Paged attention, virtual-memory KV cache
- LLMEngine, AsyncLLMEngine
- Multi-LoRA serving, S-LoRA
- Alternative to SGLang, TGI (Text Generation Inference), TensorRT-LLM, MAX
- Continuous batching = iteration-level scheduling = dynamic batching (LLM context)
