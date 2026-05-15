# Inferless

> Serverless GPU inference platform for deploying ML models

| Field | Value |
|-------|-------|
| Name | Inferless |
| Group | Compute & GPU Infrastructure |
| Type | API/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Docs | [docs.inferless.com](https://docs.inferless.com/) |

## Overview

Inferless is a serverless GPU inference platform that deploys custom machine learning models with minimal cold start latency. The platform abstracts away infrastructure management, GPU provisioning, and container orchestration so teams can focus on model development. Inferless supports importing models from HuggingFace, GitHub, GitLab, AWS S3, Google Cloud Storage (GCS), DockerHub, Dockerfiles, and direct file uploads. The billing model charges per second of actual compute usage rather than reserved capacity, with fractional GPU support so multiple models and workloads can share GPUs with automatic rebalancing and node draining [1][2].

The platform supports PyTorch, TensorFlow, ONNX, and custom Python functions without framework restrictions. It provides a web dashboard, a Command-Line Interface (CLI), a Python client library, and REST API endpoints for managing deployments. Built-in Prometheus metrics and Grafana dashboards track GPU utilization and system performance. Auto-scaling handles traffic from zero to thousands of GPUs based on requests per second [1][3].

## Core Concepts

- **Serverless GPU Inference**: Models run on GPU-backed infrastructure without users managing servers, containers, or orchestration layers. Resources are provisioned on demand and released when idle, following a pay-per-second billing model [2].
- **Model Import**: Inferless supports seven integration methods for importing models: HuggingFace, GitHub/GitLab custom code, file upload, AWS S3, Google Cloud Buckets, DockerHub, and Dockerfile-based deployments [4].
- **Autoscaling**: The platform automatically scales endpoints from zero instances up to a configured maximum based on requests per second. Min scale defines workers kept continuously running; max scale sets the maximum concurrent workers allowed [5].
- **Shared and Dedicated GPU Instances**: Shared instances allow fractional GPU access (partial GPU memory and vCPUs) at lower cost, while dedicated instances provide full GPU resources with higher memory and compute allocations [6].
- **NFS Volumes**: NFS-like writable volumes enable simultaneous connections across multiple replicas for storing model parameters, archiving datasets centrally, and establishing shared caches [7].
- **Custom Runtimes**: YAML-based configuration specifies CUDA version, Python packages, system packages, and shell commands for building custom container environments without complex application server code [8].
- **Dynamic Batching**: Inference requests are combined server-side into batches to improve throughput. Configurable via `BATCH_SIZE` and `BATCH_WINDOW` parameters in the input schema [9].
- **Streaming Output**: Server-Sent Events (SSE) enable one-way server-to-client communication for real-time data streaming during inference, with automatic reconnection after connection loss [10].
- **Secrets Manager**: Centralized storage for passwords, API keys, and tokens with encryption at rest and in transit, access control, and automatic rotation support [11].
- **Remote Run**: Execute code on remote GPU servers directly from a local machine using annotations and the `inferless remote-run` command, supporting T4, A10, and A100 GPUs [12].

## Architecture

Inferless follows a serverless architecture pattern for GPU inference with these primary components:

```
Model Source              Inferless Platform              Client
(HuggingFace,       -->  [ Import & Build Layer    ]
 S3, GCS, GitHub,        [ Custom Runtime Builder   ]
 DockerHub,              [ GPU Scheduling Engine    ]
 Dockerfile,             [ Autoscaler (0..N GPUs)   ]  -->  REST API / SSE
 File Upload)            [ Dynamic Batcher          ]
                         [ NFS Volume Storage       ]
                         [ Secrets Manager          ]
                         [ Prometheus + Grafana     ]
```

- **Import and Build Layer**: Fetches model artifacts from the configured source, builds the container with specified runtime dependencies, and prepares the model for deployment. Build progress is tracked with streaming logs via WebSockets [13].
- **Custom Runtime Builder**: Constructs container images from YAML configuration specifying CUDA version (12.4.1, 12.1.1, or 11.8.0), system packages, Python packages, and shell commands. Supports runtime versioning with in-place updates [8][13].
- **GPU Scheduling Engine**: Allocates GPU resources (T4, A10, A100) to inference requests. Supports both shared instances (fractional GPU) and dedicated instances (full GPU) [6].
- **Autoscaler**: Monitors request traffic and scales instances between zero and the configured maximum. Integrates warm pools to minimize cold start delays during scaling events [5][13].
- **Dynamic Batcher**: Combines concurrent inference requests into batches based on configurable batch size and time window, improving throughput for stateless models [9].
- **NFS Volume Storage**: Persistent shared storage accessible across multiple replicas at `/var/nfs-share/<volume-name>`. Temporary storage available at `/tmp` (deleted when model stops) [7][14].
- **Monitoring Layer**: Built-in Prometheus metrics and Grafana dashboards for GPU utilization, latency tracking, and system performance observability [1].

## Key Features

- **Per-Second Billing**: Charges based on total seconds models are running in a healthy state, rounded up to the nearest second. Billing components include model weight loading time, inference duration, and eviction timeout (5 seconds to 60 minutes of warm status) [6].
- **Scale-to-Zero**: Endpoints scale down to zero instances when idle, eliminating costs during periods of no traffic. Scale-down timing is configurable to balance cost savings against cold start latency [15].
- **Seven Integration Sources**: HuggingFace, GitHub/GitLab, file upload, AWS S3, Google Cloud Buckets, DockerHub, and Dockerfile imports [4].
- **Container Concurrency**: Configure 1 to 100 simultaneous requests per container, with sequential processing or batch processing modes [16].
- **Automatic Builds**: Webhook-driven automatic rebuilds when model sources update, supporting GitHub branch triggers and HuggingFace repo update webhooks [17].
- **Version Management**: Complete build history tracking with automatic version deployment. Models remain on their current version unless explicitly updated [18].
- **AWS SNS Alerts**: Integration for notifications on non-200 HTTP responses and high inference latency (exceeding 20 seconds) [19].
- **AWS PrivateLink**: Private endpoint connectivity for secure, non-public-internet traffic between AWS infrastructure and Inferless deployments [20].
- **Cold Start Optimization**: Custom-built orchestration engine, advanced router, and proprietary storage infrastructure minimize cold start latency, with initialization times as low as 3 seconds [2].
- **Remote Run**: Execute code on remote GPU servers from local machines using `inferless remote-run app.py -c config.yaml`, with automatic file transfer (up to 10MB) excluding `.git`, `*.pyc`, and `__pycache__` [12].
- **Cookbooks**: Pre-built deployment recipes for common use cases including PDF Q&A systems, voice chatbots, logo generators, ComfyUI API deployments, debugger agents, and MCP-based Google Maps agents [20].

## Use Cases

- **ML Model Serving**: Deploying trained models from HuggingFace, custom repositories, or cloud storage as production API endpoints with sub-second cold starts and automatic scaling [2].
- **Large Language Model (LLM) Inference**: Hosting open-weight LLMs (Llama, Qwen, DeepSeek, Mistral, Gemma, Phi, Mixtral) with streaming SSE output and dynamic batching for throughput optimization [10][20].
- **Variable-Traffic Workloads**: Applications with unpredictable or bursty inference demand that benefit from scale-to-zero and request-based autoscaling without idle GPU costs [5].
- **CI/CD Model Deployment**: Automatic rebuilds triggered by GitHub pushes or HuggingFace webhooks, enabling continuous deployment pipelines without manual infrastructure management [17].
- **Multi-Model Management**: Organizations deploying multiple models that need centralized endpoint management, shared NFS volumes for model weights, and unified monitoring dashboards [7].
- **Audio and Video Processing**: Transient workloads using `/tmp` storage for intermediate files during audio/video inference pipelines, with NFS volumes for persistent output storage [14].
- **AI Agent Infrastructure**: Serverless backend for AI agents requiring on-demand GPU compute, demonstrated in cookbooks for debugger agents and MCP-based tool agents [20].

## API Reference

**REST API Endpoint Pattern:**

After deploying a model, Inferless provides a unique API endpoint accessible via the dashboard's API tab. Authentication uses Workspace API keys managed through workspace settings.

```bash
curl -X POST "https://<your-endpoint-url>/v1/predict" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <workspace-api-key>" \
  -d '{
    "input": {
      "prompt": "Explain serverless inference in one sentence."
    }
  }'
```

**Python Client (Synchronous):**

```python
import inferless

result = inferless.call(
    url="https://<your-endpoint-url>",
    workspace_key="<workspace-api-key>",
    data={"prompt": "Hello, world!"}
)
```

**Python Client (Asynchronous):**

```python
import inferless

def on_complete(error, response):
    if error:
        print(f"Error: {error}")
    else:
        print(f"Result: {response}")

inferless.call_async(
    url="https://<your-endpoint-url>",
    workspace_key="<workspace-api-key>",
    data={"prompt": "Hello, world!"},
    callback=on_complete
)
```

**CLI Commands:**

```bash
# Scaffold a demo project
inferless scaffold --demo

# Initialize a model
inferless init --name <modelname>

# Deploy with GPU selection
inferless deploy --gpu T4

# Deploy with region and runtime
inferless deploy --gpu t4 --region <region> --runtime <runtime_name>

# Deploy with volume mount
inferless deploy --gpu t4 --volume <volume_name> --volume-mount-path <path>

# Remote run on GPU
inferless remote-run app.py -c config.yaml

# Volume management
inferless volume create --name <volume_name>
inferless volume cp --source <local_path> --destination <remote_path>
inferless volume ls
inferless volume rm
inferless volume list
inferless volume select --id <volume_id>

# Runtime management
inferless runtime list
inferless runtime upload
inferless runtime patch
inferless runtime version-list
```

**Input Schema Definition (`input_schema.py`):**

```python
INPUT_SCHEMA = {
    "prompt": {
        "datatype": "STRING",
        "required": True,
        "shape": [1],
        "example": ["There is a fine house in the forest"]
    },
    "num_steps": {
        "datatype": "INT8",
        "required": False,
        "shape": [1],
        "example": [50]
    },
}
```

Supported datatypes: STRING, BOOL, INT8, INT16, INT32, INT64, FP16, FP32, FP64, UINT8, UINT16, UINT32, UINT64, BYTES, BF16. Shape `[1]` returns a single variable; arrays greater than 1 return arrays; `-1` indicates variable length [21].

**Output Format:**

Outputs are returned as dictionaries from the `infer()` function without explicit schema configuration:

```python
# Simple output
return {"label_1": 0.398, "label_2": 0.563}

# Array output
return {"generated_images_base64": [img_str1, img_str2]}

# Dynamic keys (JSON stringified)
return {"result": json.dumps({"label_x": 0.4554})}
```

## Configuration

**Runtime Configuration (`inferless_runtime_config.yaml`):**

```yaml
cuda_version: "12.4.1"  # Options: "12.4.1", "12.1.1", "11.8.0" (default: 12.1.1)
system_packages:
  - libssl-dev
  - opencv
  - ffmpeg
python_packages:
  - transformers==4.41.1
  - torch==2.1.2
  - numpy
  - pandas
run_commands:
  - ln -sf /usr/lib/x86_64-linux-gnu/libcuda.so /usr/lib/libcuda.so
```

**Model App Structure (`app.py`):**

```python
class InferlessPythonModel:
    def initialize(self):
        """Load model weights and initialize pipeline (runs once on cold start)."""
        from transformers import pipeline
        self.generator = pipeline("text-generation", model="EleutherAI/gpt-neo-125M", device=0)

    def infer(self, inputs):
        """Run inference on input data (runs per request)."""
        prompt = inputs["prompt"]
        result = self.generator(prompt, max_length=100)
        return {"generated_txt": result[0]["generated_text"]}

    def finalize(self):
        """Cleanup resources (runs on shutdown)."""
        pass
```

**Dynamic Batching Configuration:**

```python
# In input_schema.py
BATCH_SIZE = 4
BATCH_WINDOW = 5000  # milliseconds

INPUT_SCHEMA = {
    "prompt": {
        "datatype": "STRING",
        "required": True,
        "shape": [1],
        "example": ["Hello"]
    },
}
```

With batching enabled, the `infer()` method receives a list of dictionaries and must return a list of dictionaries [9].

**Streaming SSE Configuration:**

```python
# In input_schema.py
IS_STREAMING_OUTPUT = True

INPUT_SCHEMA = {
    "prompt": {
        "datatype": "STRING",
        "required": True,
        "shape": [1],
        "example": ["Hello"]
    },
}
```

The `infer()` method receives a `stream_output_handler` parameter. Call `send_streamed_output()` for partial outputs and `finalise_streamed_output()` to close the stream. SSE inputs are limited to INT, STRING, and BOOLEAN datatypes with shape `[1]` [10].

**Model Settings (Dashboard):**

- **Scale Down Timeout**: Controls how quickly idle containers terminate (balance cost vs. cold start latency) [15].
- **Inference Timeout**: Maximum execution duration in seconds for inference requests [15].
- **Container Concurrency**: 1 to 100 simultaneous requests per container [16].
- **GPU Type**: T4, A10, or A100 in shared or dedicated configurations [6].
- **Min/Max Replicas**: Scaling boundaries for autoscaler [5].

## Integration Patterns

- **HuggingFace**: Direct model import via model identifier with optional webhook-based automatic rebuilds on model updates. Supports Transformer, ONNX, and custom model types [4][17].
- **GitHub/GitLab**: Deploy custom code from repositories with branch-specific automatic builds on push. Requires `app.py`, `input_schema.py`, and `inferless_runtime_config.yaml` [3][17].
- **AWS S3**: Import model artifacts from S3 buckets using AWS credentials configured through the secrets manager [4].
- **Google Cloud Storage**: Import from GCS buckets for teams using the Google Cloud ecosystem [4].
- **DockerHub**: Deploy pre-built Docker containers directly from DockerHub registries [4].
- **Dockerfile**: Build and deploy from Dockerfiles for full container customization [4].
- **AWS SNS**: Alert integration for model health monitoring with notifications on HTTP errors and latency spikes [19].
- **AWS PrivateLink**: Private endpoint connectivity for secure traffic between AWS infrastructure and Inferless [20].
- **Python Client**: Synchronous and asynchronous API calls from Python applications using the `inferless` package with Workspace API key authentication [22].
- **REST API**: Standard HTTP endpoints compatible with any programming language or HTTP client for direct inference calls [23].
- **Prometheus and Grafana**: Built-in metrics export for GPU utilization and performance monitoring integration [1].

## Examples

**Deploying from HuggingFace via Dashboard:**

1. Select "HuggingFace" from the workspace dashboard.
2. Enter model name, type (Transformer), task (Text generation), and HuggingFace model identifier.
3. Customize `app.py` and `input_schema.py` as needed.
4. Select GPU type (T4/A10/A100), set min and max replicas.
5. Configure runtime dependencies, volumes, and secrets.
6. Review and submit. Monitor build progress (typically 5-10 minutes) [5].

**Deploying from CLI:**

```bash
# Create a new project from template
inferless scaffold --demo

# Initialize with model name
inferless init --name my-llm-model

# Deploy on A10 GPU
inferless deploy --gpu A10

# Attach a volume for model weights
inferless volume create --name model-weights
inferless volume cp --source ./weights --destination /model-weights
inferless deploy --gpu A10 --volume model-weights --volume-mount-path /var/nfs-share/model-weights
```

**Calling a Deployed Endpoint (Python):**

```python
import inferless

# Synchronous call
result = inferless.call(
    url="https://your-endpoint.inferless.com",
    workspace_key="your-workspace-api-key",
    data={"prompt": "Explain serverless inference in one sentence."}
)
print(result)
```

**Streaming SSE Example (`app.py`):**

```python
class InferlessPythonModel:
    def initialize(self):
        from transformers import AutoModelForCausalLM, AutoTokenizer
        self.tokenizer = AutoTokenizer.from_pretrained("model-name")
        self.model = AutoModelForCausalLM.from_pretrained("model-name", device_map="cuda")

    def infer(self, inputs, stream_output_handler):
        prompt = inputs["prompt"]
        # Generate tokens iteratively
        for token in self.generate_stream(prompt):
            stream_output_handler.send_streamed_output({"token": token})
        stream_output_handler.finalise_streamed_output()

    def finalize(self):
        pass
```

Reference streaming template: [github.com/inferless/inferless_template_streaming](https://github.com/inferless/inferless_template_streaming) [10].

**Remote Run Example:**

```python
import inferless

@inferless.method(gpu="T4")
def generate(prompt):
    from transformers import pipeline
    generator = pipeline("text-generation", model="gpt2", device=0)
    return generator(prompt, max_length=100)
```

```bash
inferless remote-run app.py -c config.yaml
```

## Limitations

- **Closed Source**: The platform is proprietary with no self-hosted option; all inference runs on Inferless-managed infrastructure [1].
- **Vendor Lock-In**: Deployment configurations (`app.py`, `input_schema.py`, `inferless_runtime_config.yaml`) are specific to the Inferless platform and not portable to other serving solutions.
- **Cold Start Latency**: Scale-to-zero introduces cold start delays when the first request arrives after a period of inactivity, though the platform optimizes for initialization times as low as 3 seconds [2].
- **GPU Selection**: Limited to T4, A10, and A100 GPUs. No H100 or other GPU types are listed in the current pricing [6].
- **Python Version**: Remote run supports only Python 3.10; other versions may face compatibility issues [12].
- **SSE Datatype Restrictions**: Streaming output inputs are limited to INT, STRING, and BOOLEAN datatypes with shape `[1]`. Multiple inputs require JSON serialization as strings [10].
- **File Transfer Limit**: Remote run file transfer is capped at 10MB per working directory [12].
- **Secret Scope**: Secrets are user-level and can only be updated by the person who performed the model import [11].
- **Root Filesystem**: The platform restricts root file system access. Persistent storage requires NFS volumes; temporary storage at `/tmp` is deleted when the model stops [14].

## Changelog

Notable platform updates through the changelog (2023-2025):

- **June 2025**: Runtime version switching for deployed models, streaming build logs via WebSockets, improved autoscaler with warm pool integration for faster cold start recovery [13].
- **May 2025**: Continued platform performance improvements and dashboard enhancements [20].
- **April 2025**: CLI and dashboard updates [20].
- **March 2025**: Feature releases and optimizations [20].
- **February 2025**: Platform stability improvements [20].
- **January 2025**: New year platform updates [20].
- **December 2024**: End-of-year feature releases [20].
- **November 2024**: Two update cycles with infrastructure improvements [20].
- **October 2024**: Platform enhancements [20].
- **September 2024**: Infrastructure updates [20].
- **July 2024**: Mid-year feature releases [20].
- **June 2024**: Two update cycles [20].
- **May 2024**: Platform improvements [20].
- **April 2024**: Two update cycles [20].
- **March 2024**: Two update cycles [20].
- **February 2024**: Two update cycles [20].
- **January 2024**: Three update cycles [20].
- **December 2023**: Three update cycles marking the initial public changelog [20].

Refer to the [Inferless Changelog](https://docs.inferless.com/changelog/overview) for complete details on each release.

## Citations

- [1] What is Inferless - https://docs.inferless.com/introduction/what-is-inferless
- [2] Serverless GPUs for AI/ML Inference - https://www.inferless.com/serverless-gpu
- [3] Quickstart Guide - https://docs.inferless.com/introduction/quickstart
- [4] Integrations - https://docs.inferless.com/introduction/integrations
- [5] Deploy ML Models - https://docs.inferless.com/getting-started/deploy-ml
- [6] Pricing - https://www.inferless.com/pricing
- [7] Working with NFS Volumes - https://docs.inferless.com/concepts/working-with-nfs-volumes
- [8] Building Custom Images - https://docs.inferless.com/concepts/building-custom-images
- [9] Dynamic Batching - https://docs.inferless.com/concepts/dynamic-batching
- [10] Streaming with SSE - https://docs.inferless.com/concepts/streaming-with-sse
- [11] Managing Secrets - https://docs.inferless.com/concepts/managing-secrets-on-inferless
- [12] Remote Run - https://docs.inferless.com/concepts/remote-run
- [13] Changelog June 2025 - https://docs.inferless.com/changelog/June-2025/30th-June
- [14] Working with Files - https://docs.inferless.com/concepts/working-with-files
- [15] Model Settings - https://docs.inferless.com/api-reference/model-endpoint/configuring-the-model-settings
- [16] Processing Concurrent Requests - https://docs.inferless.com/concepts/processing-concurrent-requests
- [17] Automatic Builds - https://docs.inferless.com/concepts/setting-up-automatic-builds
- [18] Version Management - https://docs.inferless.com/api-reference/version-management
- [19] AWS SNS Alerts - https://docs.inferless.com/integrations/aws-sns/aws-sns
- [20] Inferless Documentation - https://docs.inferless.com/
- [21] Input/Output Schema - https://docs.inferless.com/concepts/configuring-the-input-output-schema
- [22] Python Client - https://docs.inferless.com/api-reference/model-endpoint/inferless-python-client
- [23] Model Endpoint - https://docs.inferless.com/api-reference/model-endpoint/model-endpoint

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- inferless
- serverless gpu inference
- scale-to-zero
- per-second billing
- huggingface deploy
- dockerfile deploy
- gcs import
- s3 import
- dockerhub import
- nfs volumes
- custom runtime
- inferless_runtime_config.yaml
- input_schema.py
- InferlessPythonModel
- dynamic batching
- batch_size
- batch_window
- streaming sse
- IS_STREAMING_OUTPUT
- remote run
- inferless deploy
- inferless scaffold
- t4 a10 a100
- shared gpu
- dedicated gpu
- aws privatelink
- aws sns alerts
- container concurrency
- automatic builds
- webhook rebuild
- prometheus grafana

### Verb-Noun Tasks

- Import a model from HuggingFace, GitHub, S3, GCS, DockerHub, or Dockerfile
- Define an `InferlessPythonModel` with `initialize`, `infer`, and `finalize`
- Configure CUDA version and Python packages via YAML
- Enable dynamic batching with `BATCH_SIZE` and `BATCH_WINDOW`
- Stream tokens with `stream_output_handler.send_streamed_output`
- Mount an NFS volume across replicas with `inferless volume create`
- Trigger automatic rebuilds on GitHub push or HuggingFace update
- Run a function on a remote T4/A10/A100 via `inferless remote-run`
- Scale a model from 0 to N replicas based on RPS
- Choose shared (fractional) or dedicated GPU instances
- Send AWS SNS alerts on non-200 responses or slow inferences
- Connect privately via AWS PrivateLink
- Track GPU utilization in built-in Grafana dashboards

### User Intent Phrases

- How do I deploy a HuggingFace model as a scale-to-zero GPU endpoint?
- I want fractional GPU sharing to reduce costs.
- How do I add streaming SSE to a local model deployment?
- How do I auto-rebuild my endpoint when I push to GitHub?
- How do I run Python code on a remote GPU from my laptop?
- I want per-second billing and scale-to-zero without provisioning anything.
- How do I share model weights across replicas using NFS?
- How do I deploy a Dockerfile-built container as a managed endpoint?
- Can I get private AWS PrivateLink connectivity to my inference endpoint?
- How do I batch concurrent requests server-side?
- How do I monitor GPU utilization with Grafana?

### Problem Statements

- I want managed serverless GPU inference without writing infra code.
- My inference traffic is bursty and idle GPUs are too expensive.
- I need a way to ship HuggingFace models behind an API in minutes.
- My current platform does not auto-rebuild on model updates.
- I want fractional GPU sharing for small models that do not need a full A100.
- I need PrivateLink/PrivateEndpoint connectivity for compliance.

### When to Pick This

- Pick this when your workload is strictly inference and scale-to-zero is paramount (vs Modal's broader compute model or Ray's general-purpose distributed compute).
- Pick this when fractional/shared-GPU instances on T4/A10/A100 match your VRAM needs and H100 is not required (vs RunPod and Modal which expose H100/H200/B200).
- Pick this when seven first-class import sources (HF, GitHub/GitLab, S3, GCS, DockerHub, Dockerfile, upload) cover your model origins.
- Pick this when YAML runtime files plus `InferlessPythonModel` are an acceptable lock-in.
- Pick this when AWS PrivateLink and SNS alerts are part of your compliance/operations.
- Pick this when automatic rebuilds on git/HF webhooks are a primary requirement.
- Pick this when dynamic server-side batching with SSE streaming is desired out of the box.

### Related Terms and Aliases

- inference-as-a-service
- managed model endpoint
- ml model hosting
- gpu autoscaling
- scale-to-zero inference
- fractional gpu
- inferless cookbook
- huggingface hosting
- automated ml deployment
- nfs shared model weights
