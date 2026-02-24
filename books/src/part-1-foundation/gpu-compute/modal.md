# Modal

| Field          | Value                                      |
|----------------|--------------------------------------------|
| **Group**      | GPU Compute & Cloud Platforms              |
| **Type**       | API/SDK/Infra                              |
| **Open Source** | No                                        |
| **GitHub**     | N/A                                        |
| **Stars**      | N/A                                        |
| **Docs**       | [Official Docs](https://modal.com/docs)    |

## Overview

Modal is a serverless GPU compute platform purpose-built for AI and machine learning workloads. It takes Python code, packages it into a container, and executes it in the cloud with automatic horizontal scaling. The platform follows a code-first approach that eliminates YAML configuration files entirely, offering sub-second cold starts, per-second billing, and multi-cloud infrastructure. Modal targets teams that need on-demand GPU access without managing infrastructure, containers, or orchestration layers.

## Core Concepts

- **App**: Top-level container that groups related functions, images, volumes, and other resources into a single deployable unit.
- **Function**: A Python function decorated with `@app.function()` that runs remotely in the cloud. Functions are the primary unit of execution and can be invoked synchronously, asynchronously, or mapped over inputs in parallel.
- **Image**: A container image definition that specifies the runtime environment for functions. Images are built incrementally using a builder pattern (e.g., `modal.Image.debian_slim().pip_install("torch")`), and layers are cached for fast rebuilds.
- **Volume**: Persistent storage that can be mounted into function containers. Volumes survive across function invocations and deployments, making them suitable for storing model weights, datasets, and checkpoints.
- **Secret**: Environment variable management for sensitive data such as API keys, database credentials, and tokens. Secrets are injected into function containers at runtime without being embedded in code or images.
- **Sandbox**: Isolated execution environments for running untrusted or experimental code with resource limits and timeouts.

## Installation and Setup

Modal requires Python 3.9 or later. Installation and authentication are handled through the CLI:

```bash
pip install modal
modal setup  # opens browser for authentication
```

The `modal setup` command creates a local token that authenticates all subsequent CLI and SDK operations. No additional configuration files are required.

Running a Modal app locally for testing:

```bash
modal run my_app.py
```

Deploying a Modal app as a persistent service:

```bash
modal deploy my_app.py
```

## Architecture

Modal operates on a serverless execution model. When a function is invoked, Modal performs the following sequence:

1. **Image resolution**: The platform checks whether the specified container image exists in its cache. If not, it builds the image from the declarative definition.
2. **Container scheduling**: A container is scheduled on available infrastructure matching the requested resources (CPU, memory, GPU type).
3. **Code injection**: The decorated function code is serialized and injected into the container at runtime.
4. **Execution**: The function runs inside the container with access to mounted volumes, secrets, and network resources.
5. **Scaling**: Additional containers are spawned automatically based on incoming request volume, scaling from zero to thousands of concurrent instances.
6. **Teardown**: Idle containers are terminated after a configurable timeout, and billing stops immediately.

The platform abstracts away container registries, orchestration systems, load balancers, and GPU drivers. Users interact exclusively through Python decorators and the Modal SDK.

## Key Features

- **Sub-second cold starts**: Containers launch in under one second through aggressive image caching and snapshot-based initialization.
- **Per-second billing**: Compute charges are measured per second of actual usage, with no minimum billing increments for idle time.
- **Web endpoints**: Functions can be exposed as HTTP endpoints using `@app.function()` combined with `@modal.web_endpoint()`, supporting REST APIs and webhook receivers.
- **Streaming responses**: Server-sent events and streaming HTTP responses are supported natively for real-time inference applications.
- **Volume mounts**: Persistent volumes can be attached to functions for reading and writing data that persists across invocations.
- **Cloud bucket integrations**: Direct mounting of S3 and GCS buckets into function containers without manual credential wiring.
- **Scheduled jobs**: Functions can be triggered on cron schedules using `@modal.Cron("0 * * * *")` or periodic intervals.
- **Secret management**: Secrets are defined once in the Modal dashboard and referenced by name in code, with automatic injection at runtime.
- **GPU health monitoring**: The platform monitors GPU health and automatically migrates workloads away from degraded hardware.
- **Preemption handling**: Functions can register callbacks to handle preemption events gracefully, saving state before container termination.
- **Multi-node training**: Distributed training across multiple GPU nodes is available in closed beta.

## Use Cases

- **Model training**: GPU-accelerated training jobs that scale from a single GPU to multi-GPU configurations without infrastructure changes.
- **Batch inference**: Processing large datasets through ML models by mapping a function over thousands of inputs in parallel.
- **Real-time inference APIs**: Deploying model serving endpoints with automatic scaling based on request volume.
- **Data preprocessing**: Running CPU or GPU-intensive data pipelines on demand without maintaining persistent compute clusters.
- **Fine-tuning**: Running fine-tuning jobs on large language models with configurable GPU types and memory.
- **Scheduled ETL**: Periodic data extraction, transformation, and loading jobs triggered by cron schedules.

## API Reference Summary

**App definition**:

```python
import modal

app = modal.App("my-app")
```

**Function decorator**:

```python
@app.function(gpu="A100", timeout=3600, memory=32768)
def my_function(input_data):
    return process(input_data)
```

**Image builder**:

```python
image = (
    modal.Image.debian_slim(python_version="3.11")
    .pip_install("torch", "transformers")
    .apt_install("ffmpeg")
)

@app.function(image=image, gpu="H100")
def inference(prompt):
    pass
```

**Volume**:

```python
volume = modal.Volume.from_name("my-volume", create_if_missing=True)

@app.function(volumes={"/data": volume})
def write_data():
    with open("/data/output.txt", "w") as f:
        f.write("result")
    volume.commit()
```

**Web endpoint**:

```python
@app.function()
@modal.web_endpoint(method="POST")
def predict(request: dict):
    return {"result": run_model(request["input"])}
```

**Parallel map**:

```python
@app.function(gpu="T4")
def process_item(item):
    return transform(item)

@app.local_entrypoint()
def main():
    items = list(range(1000))
    results = list(process_item.map(items))
```

**Secrets**:

```python
@app.function(secrets=[modal.Secret.from_name("my-api-key")])
def call_api():
    import os
    key = os.environ["API_KEY"]
```

**Scheduled function**:

```python
@app.function(schedule=modal.Cron("0 */6 * * *"))
def periodic_job():
    pass
```

## Configuration

**GPU selection**: GPUs are requested through the `gpu` parameter on the function decorator. Available GPU types and their memory:

| GPU     | Memory   | Notes                                      |
|---------|----------|--------------------------------------------|
| T4      | 16 GB    | Budget inference                           |
| L4      | 24 GB    | General purpose inference                  |
| A10     | 24 GB    | Balanced training/inference                |
| A100    | 40/80 GB | May auto-upgrade to 80 GB                 |
| L40S    | 48 GB    | Recommended for inference (cost/perf)      |
| H100    | 80 GB    | High-end training, may upgrade to H200     |
| H100!   | 80 GB    | Reserved H100 (no upgrade)                 |
| H200    | 141 GB   | Large model training                       |
| B200    | 192 GB   | Next-gen training                          |
| B200+   | 192 GB   | Opt-in for B300 access                     |
| B300    | 288 GB   | Latest generation                          |

**Multi-GPU**: Request multiple GPUs by appending a count to the GPU type string:

```python
@app.function(gpu="H100:8")  # 8x H100, up to 1,536 GB total
def distributed_training():
    pass
```

**GPU fallbacks**: Specify multiple GPU types as a prioritized list for availability:

```python
@app.function(gpu=modal.gpu.Any(["H100", "A100-80GB"]))
def flexible_training():
    pass
```

**Resource limits**: CPU, memory, and timeout are configured per function:

```python
@app.function(cpu=4, memory=65536, timeout=7200, gpu="A100")
def heavy_job():
    pass
```

## Integration Patterns

**SDKs**: Modal provides client SDKs in three languages for invoking deployed functions:

- **Python** (primary): Full SDK for defining and invoking functions, building images, and managing resources.
- **JavaScript/TypeScript**: Client SDK for invoking Modal functions from Node.js applications.
- **Go**: Client SDK for invoking Modal functions from Go services.

**Webhook integration**: Web endpoints can serve as webhook receivers for external services, processing incoming HTTP requests with GPU-backed functions.

**Pipeline chaining**: Functions can call other Modal functions directly, enabling multi-step pipelines where each stage runs on different hardware:

```python
@app.function(gpu="A100")
def generate_embeddings(text):
    return model.encode(text)

@app.function(cpu=2)
def store_results(embeddings):
    database.insert(embeddings)

@app.local_entrypoint()
def pipeline(text):
    embeddings = generate_embeddings.remote(text)
    store_results.remote(embeddings)
```

**Cloud storage**: Volumes and cloud bucket mounts provide persistent storage across function invocations, enabling workflows that accumulate state over time.

## Examples

**Basic GPU function**:

```python
import modal

app = modal.App("gpu-example")

@app.function(gpu="A100")
def train_model():
    import torch
    device = torch.device("cuda")
    tensor = torch.randn(1000, 1000, device=device)
    result = torch.matmul(tensor, tensor.T)
    return result.shape
```

**Model serving endpoint**:

```python
import modal

app = modal.App("inference-api")

image = modal.Image.debian_slim().pip_install("transformers", "torch")

@app.cls(image=image, gpu="L40S")
class ModelServer:
    @modal.enter()
    def load_model(self):
        from transformers import pipeline
        self.pipe = pipeline("text-generation", model="meta-llama/Llama-2-7b-hf", device="cuda")

    @modal.web_endpoint(method="POST")
    def generate(self, request: dict):
        result = self.pipe(request["prompt"], max_new_tokens=256)
        return {"output": result[0]["generated_text"]}
```

**Batch processing with parallel map**:

```python
import modal

app = modal.App("batch-processing")
volume = modal.Volume.from_name("results", create_if_missing=True)

@app.function(gpu="T4", volumes={"/output": volume})
def process_image(image_path: str):
    result = run_inference(image_path)
    with open(f"/output/{image_path}.json", "w") as f:
        json.dump(result, f)
    volume.commit()

@app.local_entrypoint()
def main():
    image_paths = get_all_image_paths()
    list(process_image.map(image_paths))
```

## Limitations

- **Closed source**: The platform is proprietary with no self-hosted deployment option. All workloads run on Modal-managed infrastructure.
- **Vendor lock-in**: The decorator-based API is Modal-specific. Migrating to another platform requires rewriting the infrastructure layer.
- **Multi-node training**: Distributed training across multiple nodes is in closed beta and not generally available.
- **Execution time limits**: Functions have maximum timeout constraints that may not suit extremely long-running workloads.
- **Cold start variability**: While sub-second cold starts are typical, complex images with large dependencies may take longer on first invocation.
- **Regional availability**: Infrastructure availability varies by region, which can affect GPU type availability and latency.
- **No raw VM access**: Users cannot SSH into containers or access the underlying virtual machines directly.

## Changelog Highlights

Modal is a continuously deployed platform without traditional versioned releases. The Python SDK receives frequent updates through PyPI. Notable platform capabilities include the addition of B200 and B300 GPU support, the introduction of cloud bucket mounts, and the ongoing closed beta for multi-node training.

## Citations

- [1] [Modal Documentation](https://modal.com/docs/guide)
- [2] [Modal GPU Guide](https://modal.com/docs/guide/gpu)
