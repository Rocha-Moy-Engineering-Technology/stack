# Triton Inference Server

> NVIDIA inference serving software for multi-framework model deployment

| Field | Value |
|-------|-------|
| Group | Inference Engines |
| Type | SDK/Infra |
| Open Source | Yes |
| GitHub | [https://github.com/triton-inference-server/server](https://github.com/triton-inference-server/server) |
| Stars | 10660 |
| Documentation | [Official Docs](https://docs.nvidia.com/deeplearning/triton-inference-server/) |

## Overview

Triton Inference Server is NVIDIA's open-source platform for streamlining AI model deployment and inference. It enables teams to serve models from multiple deep learning and machine learning frameworks simultaneously across diverse environments -- cloud, data center, edge devices, and embedded systems. Triton supports NVIDIA GPUs, x86 and ARM CPUs, and AWS Inferentia hardware. [1][2]

The server supports concurrent execution of multiple models, dynamic batching for optimized throughput, sequence batching for stateful models, model ensembles, and comprehensive metrics for monitoring. Communication is available via both HTTP/REST and gRPC protocols based on the KServe inference standard. The current release is version 2.65.0 (NGC container 26.01). [2]

## Core Concepts

### Model Repository

The model repository is a directory structure containing all models available for serving. Each model has a versioned directory with the model artifact and a configuration file (`config.pbtxt`) defining the model's inputs, outputs, and serving parameters. Triton monitors the repository for changes and can load/unload models dynamically. [1]

### Backends

Backends are framework-specific inference engines that execute models. Triton provides built-in backends for:

- **TensorRT** -- NVIDIA's optimized inference engine
- **PyTorch** (LibTorch) -- PyTorch model execution
- **ONNX Runtime** -- Cross-platform ONNX model inference
- **OpenVINO** -- Intel's inference optimization toolkit
- **Python** -- Custom Python-based model logic
- **RAPIDS FIL** -- Forest inference for tree-based ML models [2]

### Dynamic Batching

Triton automatically combines individual inference requests into batches to maximize GPU throughput. The scheduler holds requests briefly (configurable delay) to accumulate a batch, balancing latency and throughput. [2]

### Sequence Batching

For stateful models (e.g., recurrent networks, conversational AI), sequence batching groups related requests and maintains implicit state across a sequence of inference calls. [2]

### Model Ensembles

Ensembles chain multiple models into a pipeline within Triton, where the output of one model feeds into the next. This enables complex inference workflows (preprocessing -> inference -> postprocessing) without external orchestration. [2]

### Business Logic Scripting (BLS)

BLS enables Python-based orchestration of multiple models within a single request, supporting conditional execution, loops, and complex routing logic beyond what static ensembles provide. [2]

## Architecture

### Server Architecture

- **HTTP/gRPC Frontend** -- Handles client connections and protocol translation
- **Scheduler** -- Manages dynamic batching, sequence batching, and request queuing
- **Model Manager** -- Loads/unloads models, monitors repository for changes
- **Backend Framework** -- Pluggable backend system for framework-specific execution
- **Memory Manager** -- GPU and CPU memory allocation for input/output tensors
- **Metrics Collector** -- Prometheus-compatible metrics for GPU utilization, latency, and throughput

### Model Repository Layout

```
model_repository/
  model_a/
    config.pbtxt
    1/
      model.plan          # TensorRT
    2/
      model.plan          # Version 2
  model_b/
    config.pbtxt
    1/
      model.onnx          # ONNX Runtime
  ensemble_model/
    config.pbtxt
    1/
      <empty>             # Ensemble definition in config
```
[1]

### Inference Pipeline

1. Client sends request via HTTP or gRPC
2. Frontend parses and validates request
3. Scheduler batches request with others (dynamic batching)
4. Backend executes model inference on GPU/CPU
5. Results returned through the same path
6. Metrics recorded for monitoring [1]

## Key Features and Functionality

### Multi-Framework Concurrent Serving

Serve TensorRT, PyTorch, ONNX, and Python models simultaneously on the same GPU, with Triton managing resource allocation and scheduling. [2]

### Dynamic Batching

```
# config.pbtxt
dynamic_batching {
  preferred_batch_size: [4, 8]
  max_queue_delay_microseconds: 100
}
```

Automatically combines requests into batches with configurable preferred sizes and maximum wait times. [1]

### Model Versioning

Serve multiple versions of the same model simultaneously. Version policies control which versions are active:

```
# config.pbtxt
version_policy: { latest { num_versions: 2 } }
```
[1]

### GPU Metrics

Triton exposes Prometheus-compatible metrics including GPU utilization, inference request count, batch sizes, queue times, and per-model latency percentiles. [1]

### Decoupled Models

Support for asynchronous, multi-response models where a single request can generate multiple responses over time, enabling streaming and event-driven architectures. [2]

### Repository Agents

Extensible agents that intercept model loading to perform authentication, decryption, decompression, or format conversion during the model load process. [2]

### Custom Backends

Develop new backends in C/C++ or Python to support additional model formats or custom inference logic. [2]

## Use Cases

### Multi-Model Production Serving

Deploy dozens of models from different frameworks on shared GPU infrastructure, maximizing hardware utilization while maintaining per-model SLAs. [1]

### Real-Time Inference Pipelines

Use model ensembles to build preprocessing -> inference -> postprocessing pipelines that execute entirely within Triton, minimizing inter-service latency. [2]

### Edge Deployment

Deploy optimized TensorRT models on edge devices using Triton's in-process C API for low-latency, embedded inference. [2]

### A/B Testing

Serve multiple model versions simultaneously and route traffic between them for online evaluation. [1]

## API Reference Summary

### HTTP/REST Endpoints

- `GET /v2/health/ready` -- Server readiness check
- `GET /v2/health/live` -- Server liveness check
- `GET /v2/models/<name>` -- Model metadata
- `POST /v2/models/<name>/infer` -- Inference request
- `POST /v2/models/<name>/load` -- Load model
- `POST /v2/models/<name>/unload` -- Unload model
- `GET /v2/models/<name>/config` -- Model configuration
- `GET /metrics` -- Prometheus metrics

### gRPC Services

- `ServerLive` / `ServerReady` -- Health checks
- `ModelInfer` -- Inference execution
- `ModelReady` -- Model readiness
- `ModelMetadata` -- Model information
- `RepositoryModelLoad` / `RepositoryModelUnload` -- Dynamic model management [1]

### Client Libraries

- **Python**: `tritonclient[http]` and `tritonclient[grpc]`
- **Java**: In-process API
- **C++**: In-process API [1]

## Configuration and Customization

### Model Configuration (config.pbtxt)

- **`platform`** / **`backend`** -- Inference backend selection
- **`max_batch_size`** -- Maximum batch size (0 disables batching)
- **`input`** / **`output`** -- Tensor names, data types, and shapes
- **`instance_group`** -- GPU/CPU placement and instance count
- **`dynamic_batching`** -- Batching scheduler configuration
- **`sequence_batching`** -- Stateful model configuration
- **`ensemble_scheduling`** -- Pipeline model composition
- **`version_policy`** -- Active version management
- **`optimization`** -- TensorRT optimization settings

### Server Options

- **`--model-repository`** -- Path to model repository
- **`--model-control-mode`** -- Model management (none, poll, explicit)
- **`--strict-model-config`** -- Require explicit config files
- **`--rate-limit`** -- Rate limiting mode
- **`--http-port`** / **`--grpc-port`** -- Server port configuration
- **`--metrics-port`** -- Prometheus metrics port
- **`--log-verbose`** -- Logging verbosity level [1]

## Integration Patterns

### With TensorRT

Optimize models with TensorRT before deploying on Triton for maximum NVIDIA GPU performance.

### With KServe

KServe uses Triton as a ServingRuntime for Kubernetes-native model deployment with autoscaling.

### With Kubernetes

Deploy via Helm charts or NVIDIA GPU Operator for production Kubernetes clusters.

### With Prometheus/Grafana

Native Prometheus metrics export for dashboards and alerting.

### With NVIDIA RAPIDS

Combine Triton inference with RAPIDS for end-to-end GPU-accelerated ML pipelines.

## Examples

### Serve an ONNX Model

```
# model_repository/resnet50/config.pbtxt
name: "resnet50"
backend: "onnxruntime"
max_batch_size: 8
input [{
  name: "input"
  data_type: TYPE_FP32
  dims: [3, 224, 224]
}]
output [{
  name: "output"
  data_type: TYPE_FP32
  dims: [1000]
}]
```

```bash
tritonserver --model-repository=/models
```
[1]

### Python Client

```python
import tritonclient.http as httpclient
import numpy as np

client = httpclient.InferenceServerClient(url="localhost:8000")

inputs = [httpclient.InferInput("input", [1, 3, 224, 224], "FP32")]
inputs[0].set_data_from_numpy(np.random.rand(1, 3, 224, 224).astype(np.float32))

result = client.infer("resnet50", inputs)
output = result.as_numpy("output")
```
[1]

## Limitations and Considerations

- **NVIDIA ecosystem** -- Optimized for NVIDIA GPUs; CPU and non-NVIDIA hardware support is secondary
- **Configuration complexity** -- Model configuration via protobuf text format has a steep learning curve
- **Container size** -- Docker images are large (multi-GB) due to bundled frameworks
- **Memory management** -- Multiple concurrent models compete for GPU memory; requires careful resource planning
- **Python backend overhead** -- Python-based models have higher latency than native backends (TensorRT, ONNX)
- **Model conversion** -- Best performance requires TensorRT conversion, which adds a build step [1][2]

## Changelog Highlights

- **v2.65.0** -- Latest release (NGC container 26.01)
- **Business Logic Scripting** -- Python orchestration of multi-model workflows
- **Decoupled models** -- Asynchronous multi-response inference
- **Repository agents** -- Extensible model loading interceptors
- **In-process API** -- C and Java APIs for embedded deployment
- **Dynamic batching** -- Automatic request batching for throughput optimization
- **KServe V2 protocol** -- Standardized inference API across frameworks
- **Custom backends** -- Pluggable C++/Python backend framework [1][2]

## Citations

- [1] Triton Inference Server Documentation - <https://docs.nvidia.com/deeplearning/triton-inference-server/>
- [2] Triton Inference Server GitHub - <https://github.com/triton-inference-server/server>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

Triton Inference Server, NVIDIA Triton, tritonserver, TensorRT, ONNX Runtime, LibTorch, OpenVINO backend, RAPIDS FIL, Python backend, dynamic batching, sequence batching, model ensembles, Business Logic Scripting, BLS, decoupled models, repository agents, model repository, config.pbtxt, KServe V2 protocol, multi-framework serving, GPU inference, multi-model concurrency, tritonclient

### Verb-Noun Tasks

- Lay out a `model_repository/` with versioned subfolders and `config.pbtxt`
- Serve TensorRT, PyTorch, ONNX, and Python models concurrently on one GPU
- Configure dynamic batching with preferred batch size and queue delay
- Stand up an HTTP and gRPC inference endpoint with `tritonserver`
- Compose preprocessing → inference → postprocessing as a model ensemble
- Orchestrate multi-model workflows with Business Logic Scripting
- Manage model versions via `version_policy` and explicit load/unload
- Stream multiple responses with decoupled models
- Hook repository agents for auth/decryption during model load
- Scrape Prometheus metrics for GPU and per-model latency
- Run Triton as a KServe `ServingRuntime`

### User Intent Phrases

- "How do I serve multiple ML frameworks on the same GPU?"
- "What's NVIDIA's standard inference server?"
- "How do I deploy a TensorRT model in production?"
- "How do I batch many small inference requests automatically?"
- "I need preprocessing and postprocessing baked into the inference call"
- "How do I serve multiple model versions for A/B tests?"
- "How do I run ONNX, PyTorch, and Python models behind one endpoint?"
- "What inference server has the deepest NVIDIA GPU integration?"
- "How do I get Prometheus metrics out of my inference server?"
- "How do I run inference at the edge with an in-process C API?"

### Problem Statements

- Multiple ML frameworks (TF, PyTorch, ONNX, sklearn) all need a uniform serving API
- Per-model microservices waste GPU memory by holding it idle between requests
- Manual batching code in front of a model is fragile and underperforms
- Stateful sequence models lose context when batched naively
- Preprocessing as a separate service adds network hops and latency
- Embedded/edge deployments can't afford a network call per inference

### When to Pick This

- Pick this when many models from many frameworks must share GPUs
- Pick this over vLLM/SGLang when the workload spans CV, NLP, and tabular ML (not just LLMs)
- Pick this when TensorRT optimization is the performance ceiling
- Pick this when ensembles or BLS keep pre/post-processing in-process
- Pick this when embedded/edge deployment via the in-process C/Java API is required
- Pick this as the runtime backend for KServe on NVIDIA hardware
- Skip this when only LLMs are served — vLLM/SGLang are more LLM-specific

### Related Terms and Aliases

- TensorRT Inference Server (former name), tritonserver
- NVIDIA Inference Server, NGC Triton container
- KServe V2 / Open Inference Protocol (OIP)
- Triton backend framework, Triton Python backend
- "Multi-framework model server", "GPU model server"
- Alternative to TorchServe, TensorFlow Serving, BentoML, MLflow Serving
- Not to be confused with OpenAI Triton (GPU kernel DSL)
