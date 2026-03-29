# KServe

> Kubernetes-native model serving platform for ML inference at scale

| Field | Value |
|-------|-------|
| Group | Inference Serving |
| Type | SDK/Infra |
| Open Source | Yes |
| GitHub | [https://github.com/kserve/kserve](https://github.com/kserve/kserve) |
| Stars | 5125 |
| Documentation | [Official Docs](https://kserve.github.io/website/) |

## Overview

KServe is a standardized distributed generative and predictive AI inference platform for scalable, multi-framework deployment on Kubernetes. It is a Cloud Native Computing Foundation (CNCF) incubating project that provides a unified platform for deploying both generative AI (LLMs) and predictive AI (traditional ML) models at enterprise scale. KServe abstracts the complexity of model serving infrastructure while remaining accessible for rapid prototyping. [1]

The platform supports an OpenAI-compatible inference protocol for LLM integration, GPU acceleration with optimized memory management, intelligent model caching, KV cache offloading, and request-based autoscaling. For predictive AI, it provides multi-framework support (TensorFlow, PyTorch, scikit-learn, XGBoost, ONNX), canary deployments, inference pipelines, model explainability, and scale-to-zero capability. [1]

## Core Concepts

### InferenceService

The InferenceService is KServe's primary Custom Resource Definition (CRD) for deploying models on Kubernetes. It defines the model's predictor, transformer, and explainer components in a single declarative specification. KServe manages the underlying Kubernetes resources (deployments, services, ingress) automatically. [1]

### Predictor

The predictor component handles the actual model inference. KServe supports built-in predictors for common frameworks (TensorFlow, PyTorch, scikit-learn, XGBoost, ONNX) and custom predictors using any container image. [1]

### Transformer

Transformers perform pre-processing and post-processing of inference requests. They sit between the client and predictor, enabling data transformation, feature engineering, and response formatting without modifying the model itself. [1]

### Explainer

The explainer component provides model interpretability by generating feature attributions and explanations for predictions. KServe supports built-in explainers for understanding model decisions. [1]

### InferenceGraph

InferenceGraph enables building complex inference pipelines by composing multiple InferenceServices into directed graphs. This supports model ensembles, A/B testing, multi-armed bandits, and pipeline architectures. [1]

### ServingRuntime

ServingRuntimes define reusable model serving configurations including the container image, supported model formats, and resource requirements. KServe provides built-in runtimes and supports custom runtimes. [1]

## Architecture

### System Components

- **KServe Controller** -- Manages InferenceService lifecycle, creates Kubernetes resources
- **Knative Serving** (optional) -- Provides serverless scaling, revision management, traffic splitting
- **Istio/Kourier** -- Service mesh for traffic routing, canary deployments, and authentication
- **ModelMesh** (optional) -- High-density model serving with intelligent model placement
- **ServingRuntimes** -- Container images running model servers (TorchServe, Triton, etc.)

### Request Flow

1. Client sends inference request to Ingress Gateway
2. Gateway routes to the InferenceService endpoint
3. Request passes through transformer (if configured) for pre-processing
4. Predictor performs model inference
5. Response passes through transformer for post-processing
6. Explainer generates explanations (if requested)
7. Final response returned to client [1]

### Scaling Model

KServe supports request-based autoscaling via Knative, scaling pods based on concurrent request count rather than CPU/memory. Scale-to-zero reduces costs for infrequently used models, with cold start times managed by model caching. [1]

## Key Features and Functionality

### Multi-Framework Support

Deploy models from any ML framework with built-in support for:

- TensorFlow / TensorFlow Serving
- PyTorch / TorchServe
- scikit-learn
- XGBoost
- ONNX Runtime
- Hugging Face (via custom runtimes)
- NVIDIA Triton Inference Server
- LLMs with vLLM runtime [1]

### Canary Deployments

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: my-model
spec:
  predictor:
    canaryTrafficPercent: 20
    model:
      modelFormat:
        name: sklearn
      storageUri: gs://my-bucket/model-v2
```

Gradually roll out new model versions by directing a percentage of traffic to the canary, monitoring metrics, and promoting when ready. [1]

### Scale-to-Zero

With Knative, models automatically scale to zero pods when idle and scale up when requests arrive, reducing infrastructure costs for infrequently used models. [1]

### Model Explainability

Built-in explainer components provide feature attribution for predictions, enabling model interpretability without additional infrastructure. [1]

### Monitoring and Observability

Payload logging, outlier detection, adversarial detection, and data drift detection enable production monitoring of model quality and behavior. [1]

### OpenAI-Compatible Protocol

For generative AI workloads, KServe supports the OpenAI inference protocol, enabling seamless integration with LLM clients and tooling. [1]

### GPU Acceleration

Optimized GPU memory management with KV cache offloading to CPU/disk for extended sequences, enabling efficient LLM serving. [1]

## Use Cases

### Enterprise ML Platform

Build a centralized ML serving platform where data science teams deploy models through a standardized API without managing infrastructure. KServe handles autoscaling, versioning, and monitoring. [1]

### A/B Testing and Canary Releases

Deploy multiple model versions simultaneously with traffic splitting to evaluate performance before full rollout. InferenceGraph supports complex routing patterns. [1]

### LLM Serving on Kubernetes

Serve large language models with GPU acceleration, KV cache management, and OpenAI-compatible endpoints using vLLM or Triton runtimes. [1]

### Multi-Model Serving

Use ModelMesh for high-density model serving, packing hundreds of small models onto shared GPU resources with intelligent placement and caching. [1]

## API Reference Summary

### Custom Resources

- `InferenceService` -- Primary model deployment resource
- `InferenceGraph` -- Pipeline composition resource
- `ServingRuntime` -- Reusable runtime configuration
- `ClusterServingRuntime` -- Cluster-scoped runtime configuration

### Inference Protocols

- **V1 REST** -- KServe V1 prediction protocol (`/v1/models/<name>:predict`)
- **V2 REST** -- KServe V2 inference protocol (`/v2/models/<name>/infer`)
- **gRPC** -- High-performance binary protocol
- **OpenAI** -- OpenAI-compatible chat/completions endpoints [1]

### kubectl Commands

```bash
kubectl apply -f inference-service.yaml  # Deploy model
kubectl get inferenceservice              # List deployments
kubectl describe inferenceservice <name>  # Show details
kubectl delete inferenceservice <name>    # Remove deployment
```

## Configuration and Customization

### InferenceService Spec

- **`predictor.model.modelFormat`** -- Framework (sklearn, tensorflow, pytorch, onnx, etc.)
- **`predictor.model.storageUri`** -- Model artifact location (S3, GCS, Azure, PVC)
- **`predictor.model.runtime`** -- ServingRuntime to use
- **`predictor.minReplicas`** -- Minimum pod count (0 for scale-to-zero)
- **`predictor.maxReplicas`** -- Maximum pod count
- **`predictor.containerConcurrency`** -- Requests per pod for autoscaling
- **`transformer`** -- Pre/post-processing container configuration
- **`explainer`** -- Model explainability configuration

### ServingRuntime Spec

- **`containers`** -- Container image, ports, resources
- **`supportedModelFormats`** -- Compatible model formats
- **`multiModel`** -- Enable multi-model serving
- **`protocolVersions`** -- Supported inference protocols [1]

## Integration Patterns

### With Inference Engines (vLLM, Triton, TorchServe)

KServe uses inference engines as ServingRuntimes. vLLM, Triton, and TorchServe run inside KServe-managed pods with standardized API exposure.

### With Cloud Storage (S3, GCS, Azure Blob)

Models are stored in cloud object storage and downloaded at deployment time via storage initializers.

### With Istio/Envoy

Service mesh integration provides mTLS, traffic management, rate limiting, and authentication for model endpoints.

### With Kubeflow

KServe integrates as a component of the Kubeflow ML platform for end-to-end ML workflows.

### With Prometheus/Grafana

Metrics exported for monitoring model latency, throughput, and resource utilization.

## Examples

### Deploy a scikit-learn Model

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: iris-classifier
spec:
  predictor:
    model:
      modelFormat:
        name: sklearn
      storageUri: gs://kserve-examples/models/sklearn/iris
```

```bash
kubectl apply -f iris.yaml
kubectl get inferenceservice iris-classifier
```
[1]

### Deploy an LLM with vLLM

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: llama-service
spec:
  predictor:
    model:
      modelFormat:
        name: vllm
      storageUri: s3://models/llama-3.1-8b
      resources:
        limits:
          nvidia.com/gpu: 1
```
[1]

## Limitations and Considerations

- **Kubernetes required** -- KServe only runs on Kubernetes clusters; not suitable for non-Kubernetes environments
- **Complexity** -- Full installation with Knative and Istio adds significant infrastructure complexity
- **Cold starts** -- Scale-to-zero introduces latency for the first request after idle period
- **Resource overhead** -- KServe controller, Knative, and Istio components consume cluster resources
- **Model size limits** -- Very large models may require custom storage and download configurations
- **Learning curve** -- Understanding CRDs, ServingRuntimes, and inference protocols requires Kubernetes experience [1]

## Changelog Highlights

- **CNCF Incubating** -- Accepted as CNCF incubating project
- **OpenAI protocol** -- Native LLM serving with OpenAI-compatible endpoints
- **InferenceGraph** -- Pipeline composition for model ensembles and routing
- **ModelMesh** -- High-density multi-model serving
- **GPU/KV cache** -- Optimized LLM serving with KV cache offloading
- **V2 protocol** -- Standardized inference protocol across frameworks
- **Hugging Face runtime** -- Native support for HuggingFace model serving
- **Scale-to-zero** -- Serverless model serving with Knative [1]

## Citations

- [1] KServe Documentation - <https://kserve.github.io/kserve/>
