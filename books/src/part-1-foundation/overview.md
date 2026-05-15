# Part I — Foundation & Infrastructure

This part covers the layer everything else in the book builds on: where the model lives and how callers reach it. Seven groups span the full spectrum from "managed tokens with zero infrastructure" to "raw GPUs with full control".

## What's in this part

- **LLM Providers** — hosted frontier-model APIs (OpenAI, Gemini, Claude)
- **Hosted Inference APIs** — vendor-hosted inference on custom silicon (Groq, Cerebras)
- **Inference Engines** — self-hosted serving frameworks and model libraries (Max, vLLM, SGLang, KServe, Triton Inference Server, BentoML, Hugging Face Transformers)
- **Local Model Runtimes** — developer-focused desktop and CLI runners (Ollama, LM Studio)
- **Model Gateways** — unified API proxies across LLM providers (LiteLLM, Portkey, ccapi)
- **Compute & GPU Infrastructure** — serverless GPU platforms and distributed compute (Ray, Modal, RunPod, Vast.ai, Inferless)
- **Managed AI Platforms** — cloud-vendor end-to-end ML/GenAI platforms (Vertex AI, AWS Bedrock)

## How to navigate this part

The seven groups are ordered roughly by ascending control and descending convenience. **LLM Providers** and **Hosted Inference APIs** sit at the top: you write a prompt, tokens come back, the vendor handles everything else. **Inference Engines** and **Local Model Runtimes** drop one level — you bring the model and the host, the engine provides the serving primitives. **Model Gateways** are an orthogonal layer that fronts any of the above to give applications a single API across many providers. **Compute & GPU Infrastructure** is the rawest level — you get GPUs and decide what to run on them. **Managed AI Platforms** bundle the entire stack under one vendor and one billing account.

For most Head-of-AI decisions, the question is not "which row" but "which layer" — once you've decided whether you want managed tokens, self-hosted serving, raw compute, or an end-to-end platform, the choice within the layer becomes much narrower and is dominated by cost, latency, and ecosystem fit.
