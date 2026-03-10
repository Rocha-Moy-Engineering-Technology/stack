# RunPod

> GPU cloud platform offering pod instances and serverless GPU endpoints

| Field | Value |
|-------|-------|
| Name | RunPod |
| Group | GPU Infrastructure |
| Type | API/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Docs | [docs.runpod.io](https://docs.runpod.io/) |

## Overview

RunPod is a cloud computing platform purpose-built for AI, machine learning (ML), and general compute workloads. Founded in October 2022, it provides scalable GPU and CPU resources for training, fine-tuning, and inference across 31+ global regions. The platform offers four primary products: Pods (dedicated GPU/CPU instances billed per minute), Serverless (autoscaling workers billed per second), Instant Clusters (managed multi-node GPU clusters with high-speed networking), and Public Endpoints (pre-deployed AI model APIs). RunPod emphasizes rapid deployment through preconfigured workers, container-based workflows, and managed clustering for distributed training. The platform holds SOC 2 Type II compliance and advertises 99.9% uptime with zero-egress-fee S3-compatible persistent storage.

## Core Concepts

- **Serverless Endpoints**: Request-routing access points that manage queuing and autoscaling of worker instances with pay-per-second billing and zero idle costs
- **Workers**: Container instances executing compute tasks, available as Flex Workers (scale-to-zero on-demand) or Active Workers (always-on with up to 30% discount)
- **Handler Functions**: User-defined processing logic receiving input events and returning results, forming the core unit of serverless execution for queue-based endpoints
- **Pods**: Dedicated GPU or CPU instances running containerized workloads billed by the minute, available as On-Demand, Savings Plan (3 or 6 month commitment), or Spot instances
- **Public Endpoints**: Pre-deployed API access points to AI models covering image, video, audio, and text generation without requiring custom deployment
- **Instant Clusters**: Managed multi-node clusters with 1600-3200 Gbps inter-node networking, supporting Slurm High-Performance Computing (HPC) scheduling and distributed PyTorch training across 2-8 nodes
- **FlashBoot**: Cold start optimization reducing worker startup latency through cached container images and model weights, achieving sub-200ms cold starts
- **Network Volumes**: Persistent portable storage independent of compute resources, attachable to multiple Pods or Serverless endpoints simultaneously, surviving Pod deletion
- **Container Disk**: Temporary storage existing only while a Pod or worker is running, housing the operating system and ephemeral data
- **Volume Disk**: Pod-specific persistent storage mounted at `/workspace` by default, persisting between stops but erased on Pod deletion
- **Secure Cloud**: Pods deployed in T3/T4 data centers with enterprise-grade reliability and security for production workloads
- **Community Cloud**: A peer-to-peer model connecting individual compute providers with users through a vetted security system at competitive pricing
- **GPU Pools**: Grouped GPU types for serverless endpoint deployment (e.g., AMPERE_16, ADA_24, HOPPER_141), enabling workload placement by memory tier

## Installation

RunPod is a managed cloud platform and does not require local installation. Access is provisioned through the RunPod web console, CLI, or API.

1. Create an account at [runpod.io](https://www.runpod.io/)
2. Add billing credentials and select a compute plan
3. Generate an API key from the Settings page with appropriate permissions (All, Restricted, or Read Only)
4. For serverless workloads, create an endpoint and deploy a worker container
5. For pod-based workloads, launch a pod instance with the desired GPU configuration
6. Install the RunPod Python SDK for local development and testing:

```bash
pip install runpod
```

Verify installation:

```bash
python3 -c "import runpod; print(runpod.__version__)"
```

Authenticate by setting the API key as an environment variable:

```bash
export RUNPOD_API_KEY="your_api_key_here"
```

### MCP Server Integration

RunPod provides two Model Context Protocol (MCP) servers for AI-assisted development:

- **API MCP Server** (`@runpod/mcp-server`): Manages Pods, endpoints, templates, volumes, and registries via REST API with API key authentication
- **Docs MCP Server** (`https://docs.runpod.io/mcp`): Provides searchable access to RunPod documentation without authentication

Supported clients include Claude Code, Codex CLI, Cursor, VS Code with Copilot, Claude Desktop, Windsurf, Cline, and Gemini CLI.

## Architecture

### Serverless Request Flow

RunPod's serverless architecture follows a request-driven execution model with two distinct endpoint types:

**Queue-Based Endpoints** process requests through managed queues with guaranteed execution and automatic retries:

1. **Request Arrival**: An HTTP request reaches the serverless endpoint via `/run` (async) or `/runsync` (sync)
2. **Worker Dispatch**: If no active workers are available, a cold start initializes a new container instance
3. **Queuing**: Requests queue when all active workers are occupied
4. **Handler Execution**: The handler function processes the input payload from the event object
5. **Result Return**: The computed result is returned (immediately for `/runsync`, via polling `/status` for `/run`)
6. **Worker Persistence**: The worker remains active for a configurable idle period to serve subsequent requests
7. **Auto-Shutdown**: After the idle timeout expires, Flex Workers shut down to eliminate costs

**Load Balancing Endpoints** route requests directly to worker HTTP servers without queuing, suited for real-time applications. They support custom HTTP frameworks like FastAPI or Flask, allowing developers to define their own API routes and streaming behavior.

### Pod Architecture

Each Pod is a containerized computing environment with:

- Ubuntu Linux-based container with a unique Pod ID
- Hardware allocation (vCPU, system RAM, one or more GPUs)
- Network proxy enabling web access to exposed ports via `https://[pod-id]-[port].proxy.runpod.net`
- Three-tier storage (Container Disk, Volume Disk, Network Volume)
- Connection methods: SSH, Web Proxy, JupyterLab, VSCode/Cursor IDE integration

### Instant Cluster Architecture

Clusters provision multiple GPU nodes within the same data center:

- One node designated as primary (handles external traffic on `eth0`)
- Inter-node communication via dedicated interfaces `ens1`-`ens8` at 1600-3200 Gbps
- Pre-configured environment variables including NODE_RANK for distributed communication
- Native support for NCCL (NVIDIA Collective Communications Library) configuration

## Key Features

- **Pay-Per-Second Billing**: Serverless compute charges accumulate only during active execution with no idle costs; partial seconds round up to the next full second
- **Automatic Scaling**: Worker instances scale from zero to 1000+ workers in seconds based on request volume without manual intervention
- **Cold Start Mitigation**: Multiple strategies reduce startup latency including FlashBoot (sub-200ms), cached models, and configurable minimum Active Workers
- **Container-Based Deployment**: All workloads run in Docker containers, enabling reproducible environments across development and production
- **Managed Multi-Node Clustering**: Instant Clusters provide 2-8 node GPU environments with 1600-3200 Gbps interconnects for distributed training at scale
- **30+ GPU SKUs**: Extensive hardware selection from RTX 3070 through B300, including AMD MI300X (192GB), NVIDIA H200 (141GB), and B200 (180GB)
- **S3-Compatible Storage**: Persistent network volumes with zero egress fees, accessible via standard S3 API
- **Streaming Support**: Incremental output delivery via `/stream` endpoint with up to 1 MB per chunk for real-time applications
- **Webhook Notifications**: Automatic POST notifications upon job completion with retry logic, eliminating the need for polling
- **Execution Policies**: Configurable timeouts (5 seconds to 7 days), TTL controls, and low-priority job scheduling
- **Public Endpoints**: Instant API access to popular AI models (Flux, Whisper, Qwen, WAN 2.5, Kling, SORA 2) without deploying custom infrastructure
- **Fine-Tuning**: Integrated fine-tuning powered by Axolotl with support for LoRA adapters, multiple dataset formats (chat_template, completion, input_output, alpaca, sharegpt), and direct deployment to Hugging Face Hub
- **API Key Permissions**: Granular access control with All, Restricted (per-endpoint), and Read Only permission levels
- **SOC 2 Type II Compliance**: Enterprise-grade security with HIPAA and GDPR compliance

## Use Cases

- **Model Inference**: Deploy trained models as serverless endpoints for on-demand prediction serving with automatic scaling from zero to thousands of workers
- **Model Training**: Use dedicated Pods or Instant Clusters with multi-GPU configurations (up to 64 GPUs across 8 nodes) for training runs
- **Fine-Tuning**: Run parameter-efficient fine-tuning jobs using Axolotl with LoRA adapters on serverless infrastructure or dedicated Pods
- **LLM Serving**: Deploy large language models using vLLM workers for high-throughput text generation with OpenAI-compatible API endpoints
- **Batch Processing**: Queue large volumes of inference requests through queue-based endpoints with configurable TTL and execution policies for asynchronous processing
- **Distributed Training**: Leverage Instant Clusters with Slurm scheduling and distributed PyTorch across high-bandwidth GPU interconnects (up to 3200 Gbps)
- **Image and Video Generation**: Access Flux, Kling, Seedance, SORA 2, and other models via Public Endpoints for text-to-image and text-to-video generation
- **Audio Processing**: Use Whisper V3 and MiniMax Speech public endpoints for transcription and text-to-speech workloads
- **Prototyping**: Use Public Endpoints for rapid experimentation across image, video, audio, and text generation models with pay-per-use pricing

## API Reference

### Authentication

All API requests require Bearer token authentication:

```
Authorization: Bearer RUNPOD_API_KEY
Content-Type: application/json
```

### Serverless Endpoint Operations

Base URL: `https://api.runpod.ai/v2/{ENDPOINT_ID}`

| Operation | Method | Path | Description |
|-----------|--------|------|-------------|
| Run Sync | POST | `/runsync` | Submit synchronous job, wait for result (max 20 MB payload, 90s default timeout) |
| Run Async | POST | `/run` | Submit async job, returns job ID (max 10 MB payload) |
| Status | GET | `/status/{job_id}` | Retrieve job status and result |
| Stream | GET | `/stream/{job_id}` | Receive incremental output chunks (max 1 MB per chunk) |
| Cancel | POST | `/cancel/{job_id}` | Stop in-progress or queued job |
| Retry | POST | `/retry/{job_id}` | Requeue failed or timed-out job |
| Purge Queue | POST | `/purge-queue` | Clear all pending jobs |
| Health | GET | `/health` | Monitor endpoint worker and queue statistics |

### Request Schema

```json
{
  "input": {
    "prompt": "Your input here"
  },
  "webhook": "https://optional-callback-url.com",
  "policy": {
    "executionTimeout": 600000,
    "lowPriority": false,
    "ttl": 86400000
  },
  "s3Config": {
    "accessId": "KEY",
    "accessSecret": "SECRET",
    "bucketName": "NAME",
    "endpointUrl": "URL"
  }
}
```

### Response Schema

```json
{
  "id": "sync-79164ff4-d212-44bc-9fe3-389e199a5c15",
  "status": "COMPLETED",
  "output": {},
  "delayTime": 824,
  "executionTime": 3391
}
```

Status values: `IN_QUEUE`, `IN_PROGRESS`, `COMPLETED`, `FAILED`, `CANCELLED`, `TIMED_OUT`.

### Rate Limits (per 10-second window)

| Operation | Requests | Concurrent |
|-----------|----------|-----------|
| `/runsync` | 2000 | 400 |
| `/run` | 1000 | 200 |
| `/status` | 2000 | 400 |
| `/stream` | 2000 | 400 |
| `/cancel` | 100 | 20 |
| `/purge-queue` | 2 | N/A |

Rate limits scale dynamically with endpoint worker count, using the higher value between base limits and worker-calculated limits.

### Result Retention

- `/runsync`: 1 minute (extendable to 5 minutes via `?wait` parameter)
- `/run`: 30 minutes post-completion
- Public Endpoint output URLs: expire after 7 days

### Handler Function Interface

```python
import runpod

def handler(event):
    input_data = event["input"]
    result = process_data(input_data)
    return result

runpod.serverless.start({"handler": handler})
```

### Public Endpoint Models

Base URL: `https://api.runpod.ai/v2/{model-id}`

| Category | Examples | Pricing |
|----------|----------|---------|
| Image | Flux Dev, Flux Schnell, Qwen Image | $0.0024-$0.02/megapixel |
| Video | WAN 2.5, Kling, Seedance, SORA 2 | $0.50/5 seconds |
| Audio | Whisper V3, MiniMax Speech | $0.05/1000 characters |
| Text | Qwen3 32B, IBM Granite | $10.00/1M tokens |

Failed generations incur no charges.

## Configuration

### Serverless Endpoint Settings

- **Active Workers**: Minimum number of always-running workers to eliminate cold starts (up to 30% discount over Flex Workers)
- **Max Workers**: Upper bound on concurrent worker instances (default limit: 5; scales with balance up to 60+ at $900)
- **Idle Timeout**: Duration a worker remains active after completing its last request before auto-shutdown
- **GPU Selection**: Choose GPU type and quantity per worker from available GPU pools (AMPERE_16 through HOPPER_141)
- **Container Image**: Docker image and registry for worker deployment (Docker Hub, GitHub Container Registry, Amazon ECR)
- **Environment Variables**: Configuration values passed to workers through endpoint settings
- **Execution Timeout**: Active runtime limit per job (5 seconds to 7 days, default 10 minutes)
- **TTL**: Total job lifespan including queue and execution (10 seconds to 7 days, default 24 hours)
- **Extra Workers**: Additional workers provisioned during traffic spikes when Docker images are cached locally (default: 2)

### Pod Configuration

- **GPU Type and Quantity**: Select from 30+ GPU SKUs including B200, H200, H100, A100, RTX 4090, MI300X
- **Cloud Type**: Secure Cloud (T3/T4 data centers) or Community Cloud (peer-to-peer at lower cost)
- **Pricing Model**: On-Demand (pay-as-you-go), Savings Plans (3/6 month commitment), or Spot (lowest cost, 5-second termination warning)
- **Storage**: Container Disk (temporary), Volume Disk (persistent at `/workspace`), Network Volume (portable, multi-Pod)
- **Custom Start Commands**: Initialization scripts executed on Pod startup
- **Exposed Ports**: HTTP and TCP ports accessible via the RunPod network proxy
- **Pod Templates**: Pre-configured setups bundling Docker images with hardware specs, network settings, and environment variables

### Instant Cluster Configuration

- **Node Count**: 2-8 nodes (16-64 GPUs) for standard configurations; larger deployments available through sales
- **GPU Type**: B200 (3200 Gbps), H200 (3200 Gbps), H100 (3200 Gbps), A100 (1600 Gbps)
- **NCCL Settings**: Network interface configuration for NVIDIA Collective Communications Library
- **Framework**: PyTorch distributed, TensorFlow, Slurm HPC, or Axolotl fine-tuning

### vLLM Worker Configuration

- `MAX_MODEL_LEN`: Maximum context length (e.g., 8192)
- `DTYPE`: Model weight precision (float16, bfloat16, float32)
- `GPU_MEMORY_UTILIZATION`: VRAM usage control (e.g., 0.95)
- `CUSTOM_CHAT_TEMPLATE`: Custom chat formatting templates
- `OPENAI_SERVED_MODEL_NAME_OVERRIDE`: Model name override for OpenAI-compatible endpoints

### Fine-Tuning Configuration (Axolotl)

Configuration lives in `/workspace/fine-tuning/config.yaml`:

- **Model Settings**: Base model selection, bf16 precision auto-detection, 8-bit weight loading
- **LoRA Adapter Settings**: Rank (r: 8), alpha scaling, target module specification
- **Dataset Configuration**: Path, type specification (chat_template, completion, input_output, alpaca, sharegpt)
- **Training Mechanics**: Micro-batch size, gradient accumulation, learning rate, sequence length

## Integration Patterns

### Direct API Integration (Python)

```python
import requests

response = requests.post(
    "https://api.runpod.ai/v2/{endpoint_id}/runsync",
    headers={"Authorization": "Bearer {api_key}"},
    json={"input": {"prompt": "Hello, world"}}
)
result = response.json()
```

### Asynchronous Job with Polling

```python
import requests
import time

endpoint_url = "https://api.runpod.ai/v2/{endpoint_id}"
headers = {"Authorization": "Bearer {api_key}"}

job = requests.post(
    f"{endpoint_url}/run",
    headers=headers,
    json={"input": {"prompt": "Generate an image"}}
).json()

while True:
    status = requests.get(
        f"{endpoint_url}/status/{job['id']}",
        headers=headers
    ).json()
    if status["status"] in ["COMPLETED", "FAILED"]:
        break
    time.sleep(2)

print(status["output"])
```

### Webhook-Based Integration

```json
{
  "input": {"prompt": "Process this"},
  "webhook": "https://your-server.com/runpod-callback"
}
```

RunPod delivers POST notifications upon completion with automatic retry logic, eliminating polling overhead.

### vLLM Worker Deployment

Deploy a vLLM-based LLM serving worker through RunPod Hub:

1. Select a Hugging Face model (e.g., `meta-llama/Llama-3.2-3B-Instruct`)
2. Deploy via RunPod Hub with the latest vLLM worker version
3. Configure `MAX_MODEL_LEN` and other environment variables
4. The endpoint supports both RunPod native API and OpenAI-compatible requests

### Vercel AI SDK Integration

The `@runpod/ai-sdk-provider` package provides type-safe integration for JavaScript/TypeScript projects with built-in streaming support for use with the Vercel AI SDK.

### Public Endpoint Consumption

Access pre-deployed models through public endpoint APIs across image generation (Flux), video generation (WAN 2.5, Kling, Seedance), audio processing (Whisper V3), and text generation (Qwen3 32B) without custom deployment.

## Examples

### Minimal Serverless Handler

```python
import runpod

def handler(event):
    input_data = event["input"]
    name = input_data.get("name", "World")
    return f"Hello, {name}!"

runpod.serverless.start({"handler": handler})
```

### Handler with Processing Delay

```python
import runpod
import time

def handler(event):
    input_data = event["input"]
    prompt = input_data.get("prompt")
    seconds = input_data.get("seconds", 0)
    time.sleep(seconds)
    return prompt

if __name__ == "__main__":
    runpod.serverless.start({"handler": handler})
```

### Local Testing

Create `test_input.json`:

```json
{"input": {"prompt": "Hey there!"}}
```

Run locally:

```bash
python handler.py
```

### Docker Packaging

```dockerfile
FROM python:3.10-slim
WORKDIR /
RUN pip install --no-cache-dir runpod
COPY handler.py /
CMD ["python3", "-u", "handler.py"]
```

Build and push:

```bash
docker build --platform linux/amd64 --tag username/serverless-worker .
docker push username/serverless-worker:latest
```

### Development Workflow

1. Write handler function with processing logic
2. Test locally using the RunPod SDK test utilities
3. Package the handler into a Docker container
4. Push the container image to a registry (Docker Hub, GitHub Container Registry, Amazon ECR)
5. Deploy the image as a serverless endpoint through the RunPod console or API
6. Monitor endpoint metrics, logs, and health via `/health`
7. Iterate on the handler and redeploy

### Fine-Tuning Workflow

1. Navigate to the Fine-Tuning section in the RunPod dashboard
2. Specify Hugging Face model and dataset IDs
3. Provide Hugging Face token for gated models
4. Select GPU instance and deploy Pod
5. Connect via JupyterLab, Web Terminal, or SSH
6. Configure `/workspace/fine-tuning/config.yaml` with model, LoRA, and training parameters
7. Run training: `axolotl train config.yaml`
8. Test inference with vLLM and LoRA module loading
9. Upload fine-tuned model to Hugging Face Hub

## Limitations

- **Cold Start Latency**: Initial requests to idle endpoints incur startup delay as containers and models load; FlashBoot achieves sub-200ms but is not instantaneous for all configurations
- **Closed Source Platform**: The orchestration and infrastructure layer is proprietary; users cannot self-host or inspect the control plane
- **GPU Availability**: Specific GPU types may have limited availability depending on demand and region; default account limits start at 5 workers
- **Vendor Lock-In**: Serverless handler functions use the RunPod SDK interface (`runpod.serverless.start`), requiring adaptation to migrate to other platforms
- **Networking Constraints**: UDP connections are unavailable (TCP/HTTP only); inter-pod and cross-cluster networking is managed by the platform with limited user control over topology
- **No Docker Compose**: RunPod manages Docker internally; Docker Compose is not supported within Pods
- **No Windows**: Only Linux-based containers are supported; Mac builds require `--platform linux/amd64`
- **Payload Size Limits**: `/runsync` accepts up to 20 MB; `/run` accepts up to 10 MB; streaming chunks are limited to 1 MB
- **Result Expiration**: `/runsync` results expire after 1 minute (5 with `?wait`); `/run` results expire after 30 minutes; Public Endpoint output URLs expire after 7 days
- **Default Spend Cap**: $80/hour across all resources; worker limits scale with account balance (5 workers at base, up to 60+ at $900 balance)

## Changelog

RunPod evolves as a managed platform with continuous updates. Notable capabilities as of early 2026 include:

- **B200 and B300 GPUs**: Latest NVIDIA Blackwell architecture GPUs with 180GB and 288GB VRAM respectively
- **AMD MI300X Support**: 192GB AMD Instinct GPU availability
- **Instant Clusters**: Multi-node distributed training with up to 3200 Gbps inter-node networking
- **FlashBoot**: Sub-200ms cold start optimization for serverless workers
- **Public Endpoints**: Zero-deployment model access for image, video, audio, and text generation including SORA 2, Kling, and Seedance
- **vLLM Worker Templates**: Streamlined LLM serving with OpenAI-compatible API endpoints
- **MCP Server Integration**: Model Context Protocol servers for AI-assisted development across Claude Code, Cursor, and other IDEs
- **Fine-Tuning via Axolotl**: Integrated fine-tuning with LoRA adapter support and multiple dataset formats
- **SOC 2 Type II, HIPAA, and GDPR Compliance**: Enterprise-grade security certifications
- **Vercel AI SDK Provider**: Type-safe JavaScript/TypeScript integration with streaming support

Consult the [official documentation](https://docs.runpod.io/) for the latest platform changes.

## Citations

- [1] RunPod Documentation - https://docs.runpod.io/
- [2] RunPod Serverless Overview - https://docs.runpod.io/serverless/overview
- [3] RunPod Pods Overview - https://docs.runpod.io/pods/overview
- [4] RunPod Platform Concepts - https://docs.runpod.io/get-started/concepts
- [5] RunPod Instant Clusters - https://docs.runpod.io/instant-clusters
- [6] RunPod Serverless Quickstart - https://docs.runpod.io/serverless/quickstart
- [7] RunPod vLLM Deployment - https://docs.runpod.io/serverless/vllm/get-started
- [8] RunPod Serverless Pricing - https://docs.runpod.io/serverless/pricing
- [9] RunPod Pods Pricing - https://docs.runpod.io/pods/pricing
- [10] RunPod Public Endpoints - https://docs.runpod.io/public-endpoints/overview
- [11] RunPod API Keys - https://docs.runpod.io/get-started/api-keys
- [12] RunPod Fine-Tuning - https://docs.runpod.io/fine-tune
- [13] RunPod MCP Servers - https://docs.runpod.io/get-started/mcp-servers
- [14] RunPod GPU Types - https://docs.runpod.io/references/gpu-types
- [15] RunPod Endpoint Operations - https://docs.runpod.io/serverless/endpoints/operations
- [16] RunPod Send Requests - https://docs.runpod.io/serverless/endpoints/send-requests
- [17] RunPod Python SDK - https://docs.runpod.io/sdks/python/overview
- [18] RunPod Public Endpoint Requests - https://docs.runpod.io/public-endpoints/requests
- [19] RunPod Pod Selection Guide - https://docs.runpod.io/pods/choose-a-pod
- [20] RunPod Homepage - https://www.runpod.io/
