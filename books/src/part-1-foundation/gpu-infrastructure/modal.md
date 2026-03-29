# Modal

> Serverless GPU compute platform for AI workloads with elastic scaling

| Field | Value |
|-------|-------|
| Name | Modal |
| Group | GPU Infrastructure |
| Type | API/SDK/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Docs | [modal.com/docs](https://modal.com/docs) |

## Overview

Modal is a serverless cloud platform purpose-built for AI and compute-intensive workloads. It takes Python code, packages it into a container, and executes it in the cloud with automatic horizontal scaling. The platform follows a code-first approach that eliminates YAML configuration files entirely, offering sub-second cold starts, per-second billing, and multi-cloud infrastructure. Modal pools capacity across all major clouds, dynamically deciding where to run code based on the best available capacity, optimizing for both high GPU availability and low cost.

Modal targets engineers and researchers who need on-demand GPU access for inference, batch processing, training, fine-tuning, and sandboxed code execution without managing infrastructure, containers, or orchestration layers. All compute jobs are containerized and virtualized using gVisor (Google's sandboxing technology), with encryption in transit (TLS 1.3) and at rest. The platform is SOC 2 Type 2 certified and HIPAA-compliant on Enterprise plans.

## Core Concepts

- **App**: Top-level container that groups one or more Functions for atomic deployment, acting as a shared namespace. Apps can be ephemeral (created via `modal run`, existing only during script execution) or deployed (persisting indefinitely via `modal deploy`). Functions within an App scale independently; if no active inputs exist, no containers run and no compute charges accrue.
- **Function**: A Python function decorated with `@app.function()` that runs remotely in the cloud. Functions are the primary unit of execution and can be invoked synchronously, asynchronously, or mapped over inputs in parallel. Each Function scales up and down independently from other Functions in the same App.
- **Image**: A container image definition specifying the runtime environment for functions. Images are built incrementally using a builder pattern with method chaining (e.g., `modal.Image.debian_slim().pip_install("torch")`). Layers are cached for fast rebuilds, and each method call creates a cacheable layer. Images run on Debian Linux with gVisor sandboxing.
- **Volume**: Persistent distributed filesystem that can be mounted into function containers. Volumes survive across invocations and deployments, suitable for model weights, datasets, and checkpoints. Volumes v2 (beta) offers unlimited file count, improved random access, concurrent writers to distinct files, and HIPAA-compliant data deletion.
- **Secret**: Secure credential management injecting environment variables into containers at runtime. Secrets support creation via dashboard (with templates for common services), CLI, `.env` files, and programmatic dictionaries. Multiple Secrets can be combined per function.
- **Sandbox**: Secure isolated containers for executing untrusted or agent-generated code with configurable resource limits, timeouts (up to 24 hours), networking, and file access. Sandboxes support named instances, tagging, snapshots, and can be referenced by ID for reuse.
- **Notebook**: Cloud-hosted GPU-backed Jupyter environments with serverless pricing, real-time multi-user collaboration, AI-powered code completion (Claude Sonnet 4.6), and support for up to 8 NVIDIA A100s or H100s per kernel.

## Architecture

Modal operates on a serverless execution model. When a function is invoked, Modal performs the following sequence:

1. **Image resolution**: The platform checks whether the specified container image exists in its cache. If not, it builds the image from the declarative definition using layer-based caching.
2. **Container scheduling**: A container is scheduled on available infrastructure matching the requested resources (CPU, memory, GPU type, region). Modal pools capacity across multiple clouds for optimal availability.
3. **Code injection**: The decorated function code is serialized and injected into the container at runtime.
4. **Execution**: The function runs inside the gVisor-sandboxed container with access to mounted volumes, secrets, cloud bucket mounts, and network resources.
5. **Scaling**: Additional containers are spawned automatically based on incoming request volume. Scaling is controlled by `max_containers`, `min_containers`, `buffer_containers`, and `scaledown_window` parameters.
6. **Teardown**: Idle containers are terminated after a configurable scaledown window (default 60 seconds, configurable from 2 seconds to 20 minutes). Billing stops immediately.

The platform abstracts away container registries, orchestration systems, load balancers, and GPU drivers. Users interact exclusively through Python decorators and the Modal SDK. All inputs and outputs traverse Modal's control plane in `us-east-1`, regardless of the specified execution region.

### Container Lifecycle

Containers are reused across multiple inputs. Lifecycle hooks enable initialization and cleanup:

- **`@modal.enter()`**: One-time initialization when a container starts (loading model weights, importing packages).
- **`@modal.exit()`**: One-time cleanup on shutdown (closing connections, saving state). Receives a 30-second grace period before forced termination. Also triggered on preemption events.
- **`@modal.build()`**: Runs during image build time for build-step logic.

## Key Features

- **Sub-second cold starts**: Containers boot in approximately one second through aggressive image caching and snapshot-based initialization. Memory Snapshots capture container state after warm-up for even faster subsequent boots.
- **Per-second billing**: Compute charges are measured per second of actual usage, with no minimum billing increments for idle time.
- **Autoscaling**: Automatic container pool management with configurable `max_containers` (upper limit), `min_containers` (warm floor), `buffer_containers` (burst headroom), and `scaledown_window` (idle timeout). Dynamic updates via `Function.update_autoscaler()` without redeployment.
- **Web endpoints**: Functions exposed as HTTP endpoints via `@modal.fastapi_endpoint` (FastAPI), `@modal.asgi_app` (ASGI), `@modal.wsgi_app` (WSGI), or `@modal.web_server` (custom). Request bodies up to 4 GiB, unlimited response sizes. WebSocket support via RFC 6455 with 2 MiB message limit.
- **Streaming responses**: Server-sent events and streaming HTTP responses for real-time inference applications via dedicated streaming endpoint decorators.
- **Volumes**: Persistent distributed filesystems with up to 2.5 GB/s bandwidth. Automatic background commits every few seconds. Volumes v2 supports unlimited files, concurrent writers, and hard-linking.
- **Cloud bucket mounts**: Direct mounting of AWS S3, Google Cloud Storage, and Cloudflare R2 buckets into function containers using AWS Mountpoint technology. Supports read-only mode, key prefix filtering, and OIDC-based authentication.
- **Scheduled jobs**: Functions triggered on cron schedules via `modal.Cron("0 * * * *")` or periodic intervals via `modal.Period(hours=5)`.
- **Secret management**: Secrets created via dashboard (with templates), CLI, `.env` files, or programmatic dicts. Environment variable injection at runtime with multiple-secret composition.
- **Sandboxes**: Secure containers for untrusted code with configurable timeouts (up to 24 hours), named instances, tagging, directory snapshots, and reusable sandbox pools.
- **Notebooks**: GPU-backed Jupyter environments with real-time collaboration, AI code completion, and serverless pricing. Automatic idle shutdown with configurable timeouts.
- **GPU health monitoring**: Automated monitoring and workload migration away from degraded hardware.
- **Preemption handling**: All functions are preemptible by default with graceful termination and automatic restart. Exit handlers execute within a grace period. Non-preemptible mode available for CPU-only functions at 3x cost multiplier.
- **Multi-node clusters (beta)**: Distributed training across up to 64 H100 SXM GPUs with 3,200 Gbps RDMA networking (RoCE protocol), gang scheduling, and rank-based coordination.
- **Cluster networking (i6pn)**: Private IPv6 networking between containers within the same workspace at 50+ Gbps bandwidth. Workspace-isolated subnets using `fdaa::/16` prefix.
- **Region selection**: Geographic placement with regions including US, EU, UK, AP, CA, SA, ME, MX, AF with pricing multipliers (1.25x for US/EU/UK/AP, 2.5x for others).
- **JavaScript/Go SDKs (beta)**: Client SDKs for invoking Modal functions and managing Sandboxes from Node.js and Go applications.
- **Integrations**: OIDC authentication, Datadog monitoring, OpenTelemetry tracing, Okta/SAML SSO, and Slack notifications.

## Use Cases

- **Real-time inference APIs**: Deploying model serving endpoints with automatic scaling, sub-second cold starts, and OpenAI-compatible API endpoints. Deploy vLLM, SGLang, or custom model servers with GPU acceleration.
- **Batch inference**: Processing large datasets through ML models by mapping a function over thousands of inputs in parallel. Hard limits of 25,000 total inputs (running + pending) and 1 million pending async spawn jobs.
- **Model training**: GPU-accelerated training jobs scaling from a single GPU to multi-GPU single-node configurations. Multi-node distributed training available in beta with RDMA networking.
- **Fine-tuning**: Running fine-tuning jobs on large language models (LoRA, full fine-tuning) with configurable GPU types and memory. Examples include Flux diffusion model and LLM fine-tuning.
- **Code sandboxing**: Executing AI-generated code, coding agents, and untrusted user code in isolated Sandboxes with resource limits, networking controls, and filesystem snapshots.
- **Data preprocessing**: Running CPU or GPU-intensive data pipelines on demand, including parallel processing of Parquet files on S3 and dataset ingestion workflows.
- **Scheduled ETL**: Periodic data extraction, transformation, and loading jobs triggered by cron schedules or interval-based periods.
- **Interactive notebooks**: GPU-backed collaborative Jupyter environments for prototyping, research, and document processing with OCR.
- **Media generation**: Image generation (Flux, Stable Diffusion), video generation (Wan2.1), music generation (ACE-Step), and speech transcription (Whisper, Kyutai STT).
- **Scientific computing**: Protein folding (Boltz-2, Chai-1, ESM3), molecular structure prediction, and other compute-intensive scientific workflows.

## API Reference

**App definition**:

```python
import modal

app = modal.App("my-app")
```

**Function decorator with GPU**:

```python
@app.function(gpu="A100", timeout=3600, memory=32768)
def my_function(input_data):
    return process(input_data)
```

**Image builder** (recommended `uv_pip_install` for faster resolution):

```python
image = (
    modal.Image.debian_slim(python_version="3.11")
    .uv_pip_install(["torch", "transformers"])
    .apt_install("ffmpeg")
    .env({"CUDA_VISIBLE_DEVICES": "0"})
)

@app.function(image=image, gpu="H100")
def inference(prompt):
    pass
```

**Class-based functions with lifecycle hooks**:

```python
@app.cls(image=image, gpu="L40S")
class ModelServer:
    @modal.enter()
    def load_model(self):
        from transformers import pipeline
        self.pipe = pipeline("text-generation", model="meta-llama/Llama-2-7b-hf", device="cuda")

    @modal.fastapi_endpoint(method="POST")
    def generate(self, request: dict):
        result = self.pipe(request["prompt"], max_new_tokens=256)
        return {"output": result[0]["generated_text"]}

    @modal.exit()
    def cleanup(self):
        del self.pipe
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

**Sandbox**:

```python
sb = modal.Sandbox.create(
    app=app,
    image=image,
    timeout=3600,
    tags={"project": "agent"}
)
process = sb.exec("python", "script.py")
print(process.stdout.read())
sb.detach()
```

**Autoscaler configuration**:

```python
@app.function(
    gpu="H100",
    min_containers=2,
    max_containers=100,
    buffer_containers=5,
    scaledown_window=120,
)
def inference(prompt):
    pass
```

**Cloud bucket mount**:

```python
bucket = modal.CloudBucketMount(
    bucket_name="my-bucket",
    secret=modal.Secret.from_name("aws-creds"),
    key_prefix="data/",
    read_only=True,
)

@app.function(volumes={"/s3": bucket})
def process_data():
    pass
```

## Configuration

### GPU Selection

GPUs are requested through the `gpu` parameter on the function decorator. Available GPU types:

- **T4** (16 GB) -- Budget inference, up to 8 per container
- **L4** (24 GB) -- General purpose inference, up to 8 per container
- **A10** (24 GB) -- Balanced training/inference, up to 4 per container (96 GB max)
- **L40S** (48 GB) -- Recommended for inference (best cost/performance trade-off), up to 8 per container
- **A100** (40 GB) -- May auto-upgrade to 80 GB at no extra cost, up to 8 per container
- **A100-40GB** (40 GB) -- Specifically 40 GB variant
- **A100-80GB** (80 GB) -- Specifically 80 GB variant
- **RTX-PRO-6000** -- Professional GPU
- **H100** (80 GB SXM) -- High-end training, may auto-upgrade to H200 at no extra cost, up to 8 per container
- **H100!** (80 GB SXM) -- Reserved H100 (no auto-upgrade to H200)
- **H200** (141 GB HBM3e, 4.8 TB/s bandwidth) -- Large model training, up to 8 per container
- **B200** (192 GB) -- NVIDIA Blackwell architecture, up to 8 per container
- **B200+** (192 GB) -- Opt-in B200 or B300 (billed as B200, B300 requires CUDA 13.0+)

### Multi-GPU

Request multiple GPUs by appending a count: `gpu="H100:8"` for 8x H100 (up to 1,536 GB total). Most GPU types support up to 8 GPUs per container; A10 supports up to 4. Requesting more than 2 GPUs typically increases wait times.

### GPU Fallbacks

Specify a prioritized list of GPU types for availability flexibility:

```python
@app.function(gpu=["H100", "A100-40GB:2"])
def run_on_80gb():
    pass
```

### Resource Limits

CPU, memory, and timeout are configured per function:

```python
@app.function(cpu=4, memory=65536, timeout=7200, gpu="A100")
def heavy_job():
    pass
```

### Region Selection

Specify execution region with pricing multipliers:

```python
@app.function(gpu="H100", region="us-east")  # 1.25x multiplier
def us_inference():
    pass
```

Available regions: `us`, `eu`, `uk`, `ap`, `ca`, `sa`, `me`, `mx`, `af`, plus sub-regions (e.g., `us-east`, `eu-west`). Broader regions improve availability and cold start times. US/EU/UK/AP incur 1.25x; CA/SA/ME/MX/AF incur 2.5x multiplier.

### Non-Preemptible Mode

CPU-only functions can opt out of preemption at 3x cost:

```python
@app.function(cpu=4, nonpreemptible=True)  # GPU not supported
def critical_job():
    pass
```

## Integration Patterns

### Client SDKs

Modal provides client SDKs in three languages:

- **Python** (primary): Full SDK for defining and invoking functions, building images, and managing all resources.
- **JavaScript/TypeScript** (beta): Client SDK via npm for invoking Modal functions, running Sandboxes, and interacting with Modal resources from Node.js applications.
- **Go** (beta): Client SDK via `go get` for invoking Modal functions and managing Sandboxes from Go services.

### Observability Integrations

- **Datadog**: Direct integration for monitoring Modal workloads.
- **OpenTelemetry**: Connect to any OTel-compatible provider for distributed tracing.
- **GPU Metrics**: Built-in GPU utilization, memory, and power draw monitoring via dashboard and `nvidia-smi`.

### Authentication and SSO

- **OIDC**: Authenticate with external services (AWS, GCP) using Modal-issued identity tokens, eliminating manual credential management.
- **Okta SSO / Custom SAML SSO**: Enterprise single sign-on integration.
- **Proxy Auth Tokens**: Authenticate web endpoint requests with `Modal-Key` and `Modal-Secret` headers.

### Pipeline Chaining

Functions call other Modal functions directly, enabling multi-step pipelines where each stage runs on different hardware:

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

### CI/CD Integration

Continuous deployment via GitHub Actions or any CI system using `modal deploy`. Modal's `modal token new` command generates service user tokens for automated deployments without browser authentication.

## Examples

**OpenAI-compatible LLM serving with vLLM**:

```python
import modal

app = modal.App("vllm-inference")

image = (
    modal.Image.debian_slim(python_version="3.11")
    .pip_install("vllm")
)

@app.cls(image=image, gpu="B200:2")
class LLMServer:
    @modal.enter()
    def start_engine(self):
        from vllm import LLM
        self.llm = LLM(model="meta-llama/Llama-3.1-8B-Instruct")

    @modal.fastapi_endpoint(method="POST")
    def generate(self, request: dict):
        from vllm import SamplingParams
        params = SamplingParams(max_tokens=256)
        outputs = self.llm.generate([request["prompt"]], params)
        return {"text": outputs[0].outputs[0].text}
```

**Batch processing with parallel map and volumes**:

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

**Sandbox for code execution agents**:

```python
import modal

app = modal.App("code-agent")
image = modal.Image.debian_slim().pip_install("numpy", "pandas")

@app.function()
def execute_user_code(code: str):
    sb = modal.Sandbox.create(
        app=app,
        image=image,
        timeout=60,
    )
    process = sb.exec("python", "-c", code)
    stdout = process.stdout.read()
    stderr = process.stderr.read()
    sb.terminate()
    return {"stdout": stdout, "stderr": stderr}
```

**Multi-node distributed training (beta)**:

```python
import modal

app = modal.App("distributed-training")

@app.function(gpu="H100:8")
@modal.clustered(n_containers=4, rdma=True)  # 32 GPUs total
def train_large_model():
    import subprocess, sys
    subprocess.run(
        ["torchrun", "--nproc_per_node=8", "train.py"],
        stdout=sys.stdout, stderr=sys.stderr, check=True,
    )
```

## Limitations

- **Closed source**: The platform is proprietary with no self-hosted deployment option. All workloads run on Modal-managed infrastructure.
- **Vendor lock-in**: The decorator-based API is Modal-specific. Migrating to another platform requires rewriting the infrastructure layer.
- **Multi-node training**: Distributed training across multiple nodes is in private beta with access requiring approval.
- **Execution time limits**: Functions have maximum timeout constraints. Sandboxes support up to 24 hours.
- **Cold start variability**: While containers boot in approximately one second, complex images with large dependencies or model loading in `@modal.enter()` may extend initialization time.
- **Regional constraints**: All inputs and outputs traverse the control plane in `us-east-1` regardless of execution region. Cluster networking (i6pn) operates within single regions only.
- **No raw VM access**: Users cannot SSH into containers or access underlying virtual machines directly.
- **Scaling limits**: Hard limits of 2,000 pending inputs, 25,000 total inputs per function, 1,000 concurrent inputs per `.map()` call, and 200 web endpoint operations/second.
- **Cloud bucket mount restrictions**: No append-mode file operations, arbitrary offset writes, file renaming, or parent directory auto-creation on mounted buckets.
- **GPU preemption**: Non-preemptible mode is not available for GPU functions; only CPU-only functions can opt out of preemption.
- **Python-first**: Defining Modal Functions remains exclusive to Python. JavaScript/TypeScript and Go SDKs (beta) support invocation and Sandbox management only.

## Changelog

Modal is a continuously deployed platform. The Python SDK receives frequent updates through PyPI. Recent highlights:

- **v1.3.5** (March 2026): `modal changelog` CLI, `Secret.update()` method, running input statistics.
- **v1.3.4** (February 2026): Directory Snapshots for Sandboxes, `Sandbox.detach()`, 8x stdin throughput improvement.
- **v1.3.3** (February 2026): Billing report API (GA), Queue/Dict `from_id()` methods, async usage warnings.
- **v1.3.2** (January 2026): Dashboard URL methods, `modal dashboard` CLI, Sandbox log support.
- **v1.3.1** (January 2026): Python 3.14t support, custom Sandbox domains, `Literal` type CLI annotations.
- **v1.3.0** (December 2025): Python 3.14 support, Python 3.9 removed, exception handling migration, `single_use_containers` parameter.
- Notable platform additions in 2025-2026: B200/B300 GPU support, RTX-PRO-6000, Volumes v2, cloud bucket mounts, Modal Notebooks, multi-node training beta, RDMA networking, and JavaScript/Go SDKs.

## Citations

- [1] [Modal Documentation - Introduction](https://modal.com/docs/guide)
- [2] [Modal GPU Acceleration Guide](https://modal.com/docs/guide/gpu)
- [3] [Modal Images Guide](https://modal.com/docs/guide/images)
- [4] [Modal Scaling Out Guide](https://modal.com/docs/guide/scale)
- [5] [Modal Volumes Guide](https://modal.com/docs/guide/volumes)
- [6] [Modal Sandboxes Guide](https://modal.com/docs/guide/sandboxes)
- [7] [Modal Web Endpoints Guide](https://modal.com/docs/guide/webhooks)
- [8] [Modal Cold Start Performance](https://modal.com/docs/guide/cold-start)
- [9] [Modal Secrets Guide](https://modal.com/docs/guide/secrets)
- [10] [Modal Cloud Bucket Mounts](https://modal.com/docs/guide/cloud-bucket-mounts)
- [11] [Modal Scheduling and Cron](https://modal.com/docs/guide/cron)
- [12] [Modal Apps and Functions](https://modal.com/docs/guide/apps)
- [13] [Modal Container Lifecycle Hooks](https://modal.com/docs/guide/lifecycle-functions)
- [14] [Modal Region Selection](https://modal.com/docs/guide/region-selection)
- [15] [Modal Notebooks Guide](https://modal.com/docs/guide/notebooks)
- [16] [Modal Security and Privacy](https://modal.com/docs/guide/security)
- [17] [Modal Multi-Node Clusters](https://modal.com/docs/guide/multi-node-training)
- [18] [Modal Preemption Guide](https://modal.com/docs/guide/preemption)
- [19] [Modal Cluster Networking](https://modal.com/docs/guide/private-networking)
- [20] [Modal JavaScript/Go SDKs](https://modal.com/docs/guide/sdk-javascript-go)
- [21] [Modal Changelog](https://modal.com/docs/reference/changelog)
