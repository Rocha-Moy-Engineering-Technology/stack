# BentoML

> Python framework for building model inference APIs and serving systems

| Field | Value |
|-------|-------|
| Group | Inference Engines |
| Type | SDK/Infra |
| Open Source | Yes |
| GitHub | [https://github.com/bentoml/BentoML](https://github.com/bentoml/BentoML) |
| Stars | 8645 |
| Documentation | [Official Docs](https://docs.bentoml.com/) |

## Overview

BentoML is a unified inference platform for deploying and scaling AI models with production-grade reliability. It allows developers to construct AI systems using Python decorators to define services, package them as "Bentos" (portable deployment units), and deploy them to any cloud environment. BentoML supports custom model deployment, GPU inference, adaptive batching, model composition, async task queues, WebSocket and streaming endpoints, and comprehensive observability. [1][2]

The platform provides both a local development experience (serving models via `bentoml serve`) and a managed cloud offering (BentoCloud) with autoscaling, CI/CD pipeline integration, and multi-region deployment via Gateways. [1]

## Core Concepts

### Service

A BentoML Service is a Python class decorated with `@bentoml.service` that defines one or more inference APIs. Each API method is decorated with `@bentoml.api` and handles incoming requests. Services manage model loading in their constructor and expose HTTP endpoints automatically. [2]

### Bento

A Bento is BentoML's packaging format that bundles the service code, model artifacts, dependencies, and configuration into a deployable unit. Bentos are versioned, reproducible, and can be deployed to any supported platform. [1]

### Runner

Runners are model inference handlers that manage the execution of model forward passes. They support adaptive batching (automatically combining individual requests into batches for GPU efficiency) and can run on separate processes or machines for scaling. [1]

### Adaptive Batching

BentoML automatically batches individual inference requests into groups for GPU processing, adapting batch sizes and wait times based on current load. This maximizes throughput without manual batching configuration. [1]

### Model Composition

Multiple models can be composed within a single service for complex inference pipelines -- for example, combining an embedding model with a classifier, or chaining preprocessing with generation. [1]

## Architecture

### System Components

- **Service** -- Python class defining APIs with model loading and inference logic
- **API Server** -- HTTP server exposing service endpoints with Swagger UI
- **Runner** -- Separate process for model execution with batching
- **Bento Builder** -- Packages service, models, and dependencies
- **BentoCloud** (optional) -- Managed deployment platform with autoscaling

### Service Architecture

```python
@bentoml.service
class MyService:
    def __init__(self):
        # Model loading happens here
        self.model = load_model()

    @bentoml.api
    def predict(self, input: str) -> str:
        # Inference logic
        return self.model(input)
```

The service runs as an HTTP server with automatic OpenAPI documentation at the root URL. [2]

### Deployment Flow

1. Define service with `@bentoml.service` and `@bentoml.api` decorators
2. Test locally with `bentoml serve`
3. Build a Bento with `bentoml build`
4. Containerize with `bentoml containerize`
5. Deploy to BentoCloud or any container platform [1][2]

## Key Features and Functionality

### Service Definition

```python
import bentoml
from transformers import pipeline

EXAMPLE_INPUT = "Breaking news: A new AI model has been released..."

@bentoml.service
class Summarization:
    def __init__(self) -> None:
        self.pipeline = pipeline('summarization')

    @bentoml.api
    def summarize(self, text: str = EXAMPLE_INPUT) -> str:
        result = self.pipeline(text)
        return f"Here's your summary: {result[0]['summary_text']}"
```
[2]

### Local Serving

```bash
bentoml serve
```

Starts the service at `http://localhost:3000` with:

- REST API endpoints for each `@bentoml.api` method
- Interactive Swagger UI for testing
- Automatic request/response validation [2]

### Python Client

```python
import bentoml

with bentoml.SyncHTTPClient("http://localhost:3000") as client:
    result = client.summarize(text="Your text here...")
    print(result)
```
[2]

### GPU Inference

BentoML natively supports GPU-accelerated inference with automatic device management:

```python
@bentoml.service(resources={"gpu": 1})
class LLMService:
    def __init__(self):
        self.model = load_model(device="cuda")
```
[1]

### Streaming and WebSocket

Support for streaming responses and WebSocket connections for real-time inference applications. [1]

### Async Task Queues

Long-running inference tasks can be offloaded to async task queues, with results polled or pushed via callbacks. [1]

### Observability

Built-in monitoring, logging, metrics (Prometheus), and distributed tracing support for production visibility. [1]

## Use Cases

### LLM Endpoints

Deploy LLMs with vLLM integration for high-throughput text generation APIs. BentoML handles batching, streaming, and scaling. [1]

### RAG Systems

Build retrieval-augmented generation pipelines combining embedding models, vector search, and generation models in a single BentoML service. [1]

### Image Generation

Deploy Stable Diffusion and other image generation models with GPU acceleration and adaptive batching. [1]

### Content Moderation

Build safety and moderation pipelines combining multiple models for content classification and filtering. [1]

## API Reference Summary

### Decorators

- `@bentoml.service` -- Define a BentoML service class
- `@bentoml.api` -- Define an API endpoint method
- `@bentoml.on_shutdown` -- Cleanup handler

### CLI Commands

- `bentoml serve` -- Start local development server
- `bentoml build` -- Build a Bento package
- `bentoml containerize` -- Create Docker image from Bento
- `bentoml list` -- List available Bentos
- `bentoml models list` -- List saved models
- `bentoml deploy` -- Deploy to BentoCloud

### Client Libraries

- `bentoml.SyncHTTPClient` -- Synchronous Python client
- `bentoml.AsyncHTTPClient` -- Asynchronous Python client [1][2]

## Configuration and Customization

### Service Configuration

```python
@bentoml.service(
    resources={"gpu": 1, "memory": "4Gi"},
    traffic={"timeout": 60, "max_concurrency": 32},
    workers=4,
)
class MyService:
    ...
```

### bentofile.yaml

```yaml
service: "service:MyService"
include:
  - "*.py"
python:
  requirements_txt: "requirements.txt"
docker:
  base_image: "python:3.11-slim"
```

### Environment Variables

- **`BENTOML_HOME`** -- BentoML data directory
- **`BENTOML_PORT`** -- Server port (default: 3000)
- **`BENTOML_CONFIG`** -- Path to configuration file [1]

## Integration Patterns

### With vLLM

BentoML wraps vLLM for production LLM serving with additional features like adaptive batching and monitoring.

### With HuggingFace Transformers

Load and serve any HuggingFace model using the Transformers pipeline API within BentoML services.

### With Triton

BentoML can orchestrate Triton Inference Server as a backend for GPU-optimized inference.

### With Kubernetes

Deploy containerized Bentos to Kubernetes with Helm charts or custom manifests.

### With BentoCloud

Managed deployment with autoscaling, A/B testing, and multi-region serving.

## Examples

### Text Summarization Service

```python
import bentoml
from transformers import pipeline

@bentoml.service
class Summarization:
    def __init__(self):
        self.pipeline = pipeline('summarization')

    @bentoml.api
    def summarize(self, text: str) -> str:
        result = self.pipeline(text, max_length=130, min_length=30)
        return result[0]['summary_text']
```

```bash
bentoml serve
curl -X POST http://localhost:3000/summarize \
  -H "Content-Type: application/json" \
  -d '{"text": "Your long article text here..."}'
```
[2]

## Limitations and Considerations

- **Python only** -- Service definitions must be in Python; no support for other languages
- **Learning curve** -- Packaging system (Bento, bentofile.yaml) requires understanding BentoML's conventions
- **Cold starts** -- Model loading during service startup can be slow for large models
- **BentoCloud dependency** -- Advanced features (autoscaling, multi-region) require the paid BentoCloud platform
- **Runner overhead** -- Separate runner processes add complexity for simple single-model deployments
- **GPU resource management** -- Multi-model GPU sharing requires careful resource configuration [1]

## Changelog Highlights

- **Unified Inference Platform** -- Consolidated serving and deployment
- **Adaptive batching** -- Automatic request batching for GPU efficiency
- **Model composition** -- Multi-model pipeline support
- **Async task queues** -- Long-running task offloading
- **WebSocket support** -- Real-time streaming inference
- **BentoCloud** -- Managed deployment platform
- **Observability** -- Built-in metrics, logging, and tracing
- **vLLM integration** -- High-throughput LLM serving [1]

## Citations

- [1] BentoML Documentation - <https://docs.bentoml.com/>
- [2] BentoML Quickstart - <https://docs.bentoml.com/en/latest/get-started/quickstart.html>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

BentoML, Bento, @bentoml.service, @bentoml.api, Runner, adaptive batching, model composition, async task queues, BentoCloud, bentofile.yaml, bentoml serve, bentoml build, bentoml containerize, SyncHTTPClient, AsyncHTTPClient, WebSocket inference, streaming inference, Python inference framework, model packaging, model serving, vLLM integration, OpenAPI auto-docs, Gateways

### Verb-Noun Tasks

- Define a service with `@bentoml.service` and `@bentoml.api` decorators
- Load the model in the service `__init__` constructor
- Serve locally with `bentoml serve` on port 3000
- Build a portable Bento with `bentoml build`
- Containerize a Bento with `bentoml containerize`
- Set resources with `@bentoml.service(resources={"gpu": 1})`
- Configure traffic with `timeout` and `max_concurrency`
- Compose multiple models into one inference pipeline
- Stream tokens or send WebSocket responses
- Offload long jobs to async task queues
- Deploy to BentoCloud for managed autoscaling
- Call the service from Python with `SyncHTTPClient`

### User Intent Phrases

- "How do I turn a Python model into a REST API?"
- "What's the most Pythonic way to serve ML models?"
- "How do I get adaptive batching without writing batching code?"
- "How do I package a model and its dependencies for deployment?"
- "How do I serve a HuggingFace pipeline behind an HTTP endpoint?"
- "What's a developer-friendly alternative to Triton for Python teams?"
- "How do I auto-generate OpenAPI docs from my inference service?"
- "How do I deploy a multi-model RAG pipeline as one service?"
- "How do I run a Stable Diffusion endpoint with GPU autoscaling?"

### Problem Statements

- Wrapping models in Flask/FastAPI re-invents batching, packaging, and observability
- Manual Docker builds for ML services drift between dev and prod
- Multi-model pipelines need glue code for in-process composition
- Long-running inference jobs (image gen, batch) need queues, not blocking HTTP
- Production-grade metrics, tracing, and logging are boilerplate to add
- Teams without Kubernetes expertise still need autoscaling and rolling deploys

### When to Pick This

- Pick this when teams are Python-first and want decorator-based service definitions
- Pick this over Triton when developer ergonomics matter more than raw multi-framework breadth
- Pick this over raw vLLM when packaging, deployment, and observability are bundled
- Pick this when model composition (embeddings + classifier, RAG) lives in one service
- Pick this when adaptive batching should be automatic, not hand-tuned
- Pick this when BentoCloud's managed autoscaling is a desired option
- Skip this when you need non-Python service definitions or maximum raw throughput

### Related Terms and Aliases

- BentoML, Bento, BentoCloud, Yatai (legacy)
- "Python model serving framework"
- "ML in production framework"
- Bento (the packaging unit) vs BentoML (the framework)
- Alternative to Cog (Replicate), Modal, Truss (Baseten), MLflow Models, Ray Serve
- Service-first model deployment, decorator-based serving
