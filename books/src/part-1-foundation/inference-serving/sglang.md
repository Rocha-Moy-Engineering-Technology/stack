# SGLang

> High-performance serving framework for large language and multimodal models

| Field | Value |
|-------|-------|
| Group | Inference Serving |
| Type | SDK |
| Open Source | Yes |
| GitHub | [https://github.com/sgl-project/sglang](https://github.com/sgl-project/sglang) |
| Stars | 23652 |
| Documentation | [Official Docs](https://docs.sglang.io/) |

## Overview

SGLang is a high-performance serving framework for large language and multimodal models, designed for low-latency and high-throughput inference across diverse hardware setups from single GPUs to large distributed clusters. The framework's core innovations include RadixAttention for efficient prefix caching and a zero-overhead CPU scheduler that maximizes GPU utilization. [1][2]

SGLang powers trillions of tokens in production daily across 400,000+ GPUs worldwide, with enterprise adoption by xAI, AMD, NVIDIA, and major cloud providers. It serves as a proven backend for reinforcement learning and post-training frameworks including AReaL, Miles, slime, Tunix, and verl. The framework supports most HuggingFace models and provides OpenAI-compatible API endpoints. [2]

## Core Concepts

### RadixAttention

RadixAttention is SGLang's key innovation for prefix caching. It uses a radix tree (compressed trie) data structure to efficiently store and retrieve KV cache entries for shared prefixes across requests. This enables automatic reuse of computed attention states when multiple requests share common prefixes (e.g., system prompts), achieving up to 5x faster inference for applicable workloads. [2]

### Zero-Overhead CPU Scheduler

SGLang's scheduler operates with zero overhead on the CPU, meaning scheduling decisions do not block GPU execution. The scheduler continuously selects the next batch of tokens to process while the GPU is executing the current batch, maximizing hardware utilization. [1][2]

### Prefill-Decode Disaggregation

SGLang separates the prefill phase (processing input tokens) from the decode phase (generating output tokens) into independent stages that can be optimized and scaled separately. This disaggregation is particularly beneficial for long-context workloads. [1]

### Continuous Batching with Paged Attention

Like vLLM, SGLang implements continuous batching (dynamically adding/removing requests from batches) combined with paged attention for efficient KV cache memory management. [1]

## Installation and Setup

### pip Install

```bash
pip install sglang[all]
```

For specific hardware:

```bash
# NVIDIA GPU
pip install sglang[all] --find-links https://flashinfer.ai/whl/cu124/torch2.5/

# AMD GPU
pip install sglang[all] --find-links https://releases.flashinfer.ai/whl/rocm/
```
[1]

### Docker

```bash
docker run --gpus all \
  -p 30000:30000 \
  -v ~/.cache/huggingface:/root/.cache/huggingface \
  lmsysorg/sglang:latest \
  python -m sglang.launch_server \
  --model-path meta-llama/Llama-3.1-8B-Instruct \
  --host 0.0.0.0 --port 30000
```
[1]

### From Source

```bash
git clone https://github.com/sgl-project/sglang.git
cd sglang
pip install -e ".[all]"
```
[2]

## Architecture

### Runtime Architecture

- **SGLang Router** -- Load balancer distributing requests across worker processes
- **SGLang Server** -- Main serving process managing model lifecycle and request handling
- **Scheduler** -- Zero-overhead CPU scheduler that batches and prioritizes requests
- **RadixAttention Cache** -- Radix tree-based KV cache for prefix sharing
- **Model Runner** -- Executes forward passes with optimized kernels (FlashInfer, Triton)
- **Tokenizer** -- Fast tokenization with HuggingFace tokenizers

### Parallelism Strategies

- **Tensor parallelism (TP)** -- Split model layers across GPUs within a node
- **Pipeline parallelism (PP)** -- Distribute model stages across nodes
- **Expert parallelism (EP)** -- Distribute MoE experts across GPUs
- **Data parallelism (DP)** -- Replicate model across groups for higher throughput [1][2]

### Hardware Support

NVIDIA GPUs (GB200, B300, H100, A100, Spark), AMD GPUs (MI355, MI300), Intel Xeon CPUs, Google TPUs, Ascend NPUs, and additional platforms. [1]

## Key Features and Functionality

### OpenAI-Compatible API Server

```bash
python -m sglang.launch_server \
  --model-path meta-llama/Llama-3.1-8B-Instruct \
  --port 30000
```

Exposes `/v1/chat/completions`, `/v1/completions`, and `/v1/embeddings` endpoints. [1]

### Speculative Decoding

Accelerate generation using draft models that predict tokens ahead for verification by the main model, reducing sequential forward passes. [1]

### Structured Outputs

Generate outputs conforming to JSON schemas, regular expressions, or context-free grammars. SGLang achieved 3x faster JSON decoding via compressed finite state machine (FSM) techniques. [2]

### Quantization

Comprehensive quantization support for memory efficiency and speed:

- FP4 and FP8 floating point
- INT4 integer quantization
- AWQ activation-aware quantization
- GPTQ post-training quantization [1]

### Multi-LoRA Batching

Serve multiple LoRA adapters simultaneously with efficient batching, enabling multi-tenant deployments from a single base model. [1]

### Chunked Prefill

Split long input sequences into chunks for prefill processing, reducing memory peaks and enabling processing of very long contexts. [1]

### Diffusion Model Support

SGLang extends beyond language models to support diffusion models including WAN video generation and Qwen image generation. [1]

## Use Cases

### High-Throughput API Serving

Deploy LLMs as production endpoints with RadixAttention providing automatic prefix caching for repeated system prompts and multi-turn conversations. [1]

### Reinforcement Learning Backend

SGLang serves as the inference backend for RL post-training frameworks (AReaL, verl, slime), providing fast rollout generation for policy training. [2]

### Large-Scale Distributed Inference

Deploy frontier-scale models across 96+ GPUs using expert parallelism and disaggregated prefill-decode, enabling inference on models like DeepSeek-V3. [1][2]

### Multimodal Applications

Serve vision-language and audio-language models with unified API endpoints for text, image, and audio inputs. [1]

## API Reference Summary

### Server Launch

- `python -m sglang.launch_server` -- Start server with model
- `--model-path` -- HuggingFace model ID or local path
- `--port` -- Server port (default: 30000)
- `--tp` -- Tensor parallelism size
- `--dp` -- Data parallelism size

### API Endpoints

- `POST /v1/chat/completions` -- Chat completions (OpenAI-compatible)
- `POST /v1/completions` -- Text completions
- `POST /v1/embeddings` -- Vector embeddings
- `POST /generate` -- Native SGLang generation endpoint
- `GET /health` -- Health check
- `GET /v1/models` -- List loaded models

### Python Client

```python
import openai

client = openai.Client(base_url="http://localhost:30000/v1", api_key="EMPTY")
response = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[{"role": "user", "content": "Hello!"}],
)
```
[1]

## Configuration and Customization

### Server Arguments

- **`--model-path`** -- Model identifier or local path
- **`--tp`** -- Tensor parallelism size (default: 1)
- **`--dp`** -- Data parallelism size (default: 1)
- **`--mem-fraction-static`** -- Static memory allocation fraction
- **`--max-running-requests`** -- Maximum concurrent requests
- **`--max-total-tokens`** -- Maximum total tokens in KV cache
- **`--context-length`** -- Override model context length
- **`--quantization`** -- Quantization method (fp8, awq, gptq)
- **`--enable-torch-compile`** -- Enable torch.compile for 1.5x speedup
- **`--chunked-prefill-size`** -- Chunk size for prefill processing
- **`--schedule-policy`** -- Scheduling policy (lpm, random, fcfs)
- **`--disable-radix-cache`** -- Disable RadixAttention caching

### Performance Tuning

- **`--enable-flashinfer`** -- Use FlashInfer attention kernels
- **`--enable-dp-attention`** -- Enable data parallelism attention
- **`--speculative-algorithm`** -- Select speculative decoding method
- **`--num-speculative-steps`** -- Number of speculative tokens [1]

## Integration Patterns

### With OpenAI SDK

SGLang's OpenAI-compatible API allows any OpenAI client to connect by changing the base URL:

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:30000/v1", api_key="EMPTY")
```

### With Agent Frameworks (LangChain, LangGraph)

Integrates as an LLM backend through the OpenAI-compatible endpoint.

### With RL Frameworks (verl, AReaL)

SGLang is the recommended inference backend for reinforcement learning post-training, providing fast rollout generation.

### With Observability (Prometheus)

Exposes metrics for monitoring throughput, latency, and GPU utilization.

## Examples

### Chat Completion

```python
import openai

client = openai.Client(base_url="http://localhost:30000/v1", api_key="EMPTY")

response = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is machine learning?"},
    ],
    temperature=0.7,
    max_tokens=256,
)
print(response.choices[0].message.content)
```
[1]

### Batch Inference with curl

```bash
curl -X POST http://localhost:30000/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "meta-llama/Llama-3.1-8B-Instruct",
    "prompt": "The capital of France is",
    "max_tokens": 32,
    "temperature": 0
  }'
```
[1]

## Limitations and Considerations

- **Hardware requirements** -- Large models require significant GPU memory; quantization reduces requirements but may affect quality
- **Cold start** -- Initial model loading and compilation takes time, especially with torch.compile enabled
- **Speculative decoding** -- Requires compatible draft models; not all model pairs yield speedups
- **Prefix caching** -- RadixAttention provides best benefits when requests share common prefixes; random workloads see less improvement
- **Model compatibility** -- Supports most HuggingFace models but not all architectures [1][2]

## Changelog Highlights

- **RadixAttention** -- Radix tree-based prefix caching (v0.1, 5x faster)
- **Compressed FSM** -- 3x faster JSON decoding for structured outputs
- **DeepSeek MLA** -- 7x speedup for MLA attention models
- **torch.compile** -- 1.5x faster execution via compilation
- **Expert parallelism** -- MoE model distribution across GPUs
- **Prefill-decode disaggregation** -- Separated processing phases
- **Diffusion support** -- WAN and Qwen image/video generation
- **Multi-LoRA batching** -- Concurrent adapter serving
- **FP4 quantization** -- Ultra-low precision inference [1][2]

## Citations

- [1] SGLang Documentation - <https://docs.sglang.io/>
- [2] SGLang GitHub Repository - <https://github.com/sgl-project/sglang>
