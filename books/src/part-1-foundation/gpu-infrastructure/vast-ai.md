# Vast.ai

> GPU marketplace for affordable on-demand cloud computing

| Field | Value |
|-------|-------|
| Name | Vast.ai |
| Group | GPU Infrastructure |
| Type | API/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Docs | [docs.vast.ai](https://docs.vast.ai/) |

## Overview

Vast.ai is a decentralized GPU marketplace that connects compute providers -- ranging from hobbyists with spare GPUs to Tier-4 datacenters -- with users who need GPU resources for AI and machine learning workloads. The platform operates on a peer-to-peer marketplace model where providers list their hardware, set their own prices, and retain full pricing autonomy through dynamic, supply-and-demand-driven pricing. GPU instances can be launched in seconds through the web console, Command Line Interface (CLI), Python Software Development Kit (SDK), or REST API.

The platform offers two primary compute paradigms: dedicated GPU instances (Docker containers or Virtual Machines (VMs) with exclusive GPU access) and a serverless inference platform that auto-scales workers behind a managed endpoint. Vast.ai's stated mission is to democratize AI compute: "compute powering AI is supplied by the people and for the people."

## Core Concepts

- **GPU Marketplace**: A peer-to-peer compute network where providers list their hardware and set their own prices. Users browse available machines, compare specs and reliability ratings, and rent GPU time at market-driven rates. Prices fluctuate based on real-time supply and demand, creating competitive rates without static price quotes.
- **Instances**: Containerized environments providing exclusive, never-shared GPU access for training, inference, and development. Each instance includes proportional CPU, RAM, and storage, runs a user-chosen Docker image, and bills by the second for actual usage. Instances come in three types: On-demand (guaranteed, fixed pricing), Reserved (up to 50% discount with commitment), and Interruptible (lowest cost, may be paused by higher-priority rentals).
- **Serverless Endpoints**: A managed inference platform that auto-scales GPU workers behind a single API endpoint. Users deploy a model (e.g., via vLLM), configure scaling parameters, and the platform handles worker provisioning, load balancing, and autoscaling based on benchmark-driven throughput metrics. Serverless supports mixed hardware -- a single endpoint can leverage diverse GPU types from consumer-grade to enterprise-class.
- **Templates**: Configuration wrappers around Docker images that simplify instance deployment. Templates encapsulate the Docker image, environment variables, startup scripts, port configuration, and deployment settings. Vast.ai provides prebuilt templates (e.g., PyTorch, vLLM) built on base images that include CUDA, Node.js, and integrated Caddy proxy with TLS encryption and authentication.
- **PyWorkers**: Custom Python worker scripts for serverless endpoints that act as HTTP proxy layers between the Vast routing system and a model server. PyWorkers handle request transformation, workload calculation, response streaming, and readiness detection through log pattern matching.
- **Search Engine**: A hardware search and filtering system allowing users to query available machines by GPU model, VRAM, CPU cores, RAM, disk space, bandwidth, provider reliability score, geographic location, and price.

## Installation

### CLI Installation

The `vastai` CLI is a self-contained Python script providing all functionality of the web console.

```bash
# Install from PyPI
pip install vastai

# Or install directly from GitHub
wget https://raw.githubusercontent.com/vast-ai/vast-python/master/vast.py -O vast
chmod +x vast
```

### Authentication

```bash
# Set API key (generated from https://cloud.vast.ai/cli/)
vastai set api-key YOUR_API_KEY
```

The API key is saved in a hidden file in the home directory. Default keys grant full account access; restricted permissions can be configured with `create api-key` and a JSON permission structure.

### Python SDK Installation

```bash
pip install vastai_sdk
```

```python
from vastai_sdk import VastAI

# Initialize with explicit key
vast_sdk = VastAI(api_key="YOUR_API_KEY")

# Or for serverless endpoints
from vastai import Serverless
client = Serverless()  # Uses VAST_API_KEY environment variable
```

## Architecture

Vast.ai follows a three-layer marketplace architecture:

- **Provider Layer**: GPU owners and datacenter operators register their machines, configure pricing and contract terms, and make hardware available to the network. Providers retain full control over pricing and availability.
- **Marketplace Layer**: The central platform handles instance discovery, search and filtering, transaction management, reliability tracking, and billing. Dynamic pricing is determined by providers, not the platform -- Vast.ai adds no markup on top of host-set prices.
- **Consumer Layer**: Users interact with the marketplace through four interfaces: the web console at cloud.vast.ai, the `vastai` CLI, the Python SDK (`vastai_sdk`), or the REST API.

### Instance Execution Environment

Instances are Linux Docker containers where templates control Docker creation parameters. The platform automatically configures resource constraints:

- **GPU**: Exclusive, never-shared access per instance. Stopped instances release GPU reservations.
- **CPU and RAM**: Scale proportionally to GPU fraction on the host. CPU can burst above baseline when spare cycles exist, but RAM overages risk Out of Memory (OOM) termination during contention.
- **Disk**: Static allocation set at creation time; cannot be modified after launch.
- **Networking**: Instances lack unique public IPs. Each open internal port maps to a random external port on shared infrastructure, with a 64-port-per-instance limit. Docker `EXPOSE` commands automatically generate port mappings; custom ports use `-p` flag syntax. Identity port mappings (matching external and internal) require ports above 70000.

Three launch modes are supported: Entrypoint (runs the Docker image's default entrypoint), SSH (injects SSH setup scripts), and Jupyter (injects Jupyter notebook setup). SSH and Jupyter modes replace the original Docker entrypoint, so users should copy their entrypoint command into the onstart script.

### Virtual Machine Instances

For workloads requiring init managers (systemd), nested containerization, Docker-in-Docker, or kernel module loading, Vast.ai offers VM instances. Pre-configured Ubuntu 22.04 Server and Ubuntu Desktop images are available. VMs have slower creation and boot times, higher disk overhead, and more limited machine availability compared to Docker instances.

### Serverless Architecture

The serverless platform provisions GPU workers behind a managed endpoint with autoscaling:

- **Endpoint**: A named API entry point that routes requests to available workers.
- **Workergroup**: A collection of GPU workers sharing the same template, model, and scaling configuration. Parameters include `gpu_ram`, `search_params`, `template_hash`, and `launch_args`.
- **Autoscaler**: Benchmark-driven scaling that identifies optimal price-performance GPUs. Workers transition between states: Stopped (model loaded, ready to activate on-demand as cold workers), Loading (starting up and loading model into GPU memory), and Ready (active and handling requests).
- **Cold Multiplier**: A scaling factor that determines total capacity (cold plus warm workers) based on predicted load.
- **Workers**: Individual GPU instances running the model server and PyWorker. The system tracks throughput per worker via benchmarks to estimate workload capacity.

## Key Features

- **Per-Second Billing**: Charged by the second for actual GPU usage, with no hourly minimums. Storage charges continue while instances exist, even when stopped.
- **Three Instance Types**: On-demand (guaranteed, highest priority), Reserved (up to 50% discount with pre-payment commitment), and Interruptible (lowest cost, may be paused, often 50% or more cheaper than on-demand).
- **Serverless Inference**: Deploy models behind auto-scaling endpoints with benchmark-driven GPU selection, mixed hardware support, and OpenAI-compatible API endpoints via vLLM templates.
- **Custom PyWorkers**: Build custom worker scripts with configurable request parsing, workload calculation, response streaming, and log-based readiness detection. Deploy via Git repository with `PYWORKER_REPO` environment variable.
- **Template System**: Prebuilt and custom templates built on Vast.ai base images (`vastai/base-image`, `vastai/pytorch`) with CUDA, integrated Caddy proxy, automatic TLS, and authentication. Templates support onstart scripts, environment variables, and port configuration.
- **Cloud Sync**: Transfer data between instances and cloud storage providers (Amazon S3, Google Drive, Dropbox, Backblaze) via GUI or `vastai cloud copy` CLI command, even when instances are stopped.
- **Instance Portal**: Web interface for instances using Vast.ai base images, providing authenticated access to services and tunnel creation without direct port exposure.
- **Team Management**: Create teams, invite members, assign roles with granular permissions, and transfer credits between personal accounts and teams.
- **Multiple Access Methods**: Connect to instances via SSH, Jupyter notebooks, web portals, or custom entrypoints.

## Use Cases

- **Model Training**: Rent high-end GPUs (A100, H100) for training large models at marketplace rates significantly below major cloud providers. Use Reserved instances for multi-day training runs to save up to 50%.
- **Inference at Scale**: Deploy serverless endpoints with auto-scaling workers for production inference. The platform handles GPU provisioning, load balancing, and scaling based on real-time demand.
- **Batch Processing**: Run large batch jobs across multiple Interruptible instances at the lowest cost. Interruptible workloads handle pauses gracefully and resume automatically when priority is restored.
- **Experimentation and Prototyping**: Quickly spin up On-demand GPU instances for short-lived experiments without long-term commitments. Per-second billing ensures minimal cost for brief sessions.
- **Fine-Tuning**: Use mid-range GPU instances with sufficient VRAM to fine-tune pretrained models on custom datasets. Templates with PyTorch and CUDA pre-installed reduce setup time.
- **Multi-Container Workloads**: Use VM instances for scenarios requiring Docker-in-Docker, Kubernetes, or systemd support that standard Docker containers cannot provide.

## API Reference

### REST API

The REST API is intended for advanced users; the CLI and Python SDK are recommended for most workflows. Authentication uses Bearer tokens with API keys generated from the web console.

### CLI Commands

```bash
# Search for available GPU offers
vastai search offers [filter-parameters] -o [sort-options]
# Example: vastai search offers 'reliability > 0.99 num_gpus>=4'

# Create an instance from an offer
vastai create instance [OFFER_ID] --image [IMAGE] --disk [GB] --ssh --direct

# List running instances
vastai show instances

# Start / stop / destroy instances
vastai start instance [ID]
vastai stop instance [ID]
vastai destroy instance [ID]

# Data transfer between instances or cloud storage
vastai copy [instance_id:]path [instance_id:]path
vastai cloud copy [args]

# SSH into an instance
vastai ssh-url [ID]

# Show underlying API call for any command
vastai [command] --explain
```

### Python SDK

```python
from vastai_sdk import VastAI
vast_sdk = VastAI(api_key="YOUR_API_KEY")

# Search for available offers
offers = vast_sdk.search_offers(query="gpu_name=RTX_5090 rented=False rentable=True")

# Launch an instance
vast_sdk.launch_instance(num_gpus="1", gpu_name="RTX_3090", image="pytorch/pytorch")

# Instance lifecycle
vast_sdk.start_instance(id=12345)
vast_sdk.stop_instance(id=12345)
vast_sdk.reboot_instance(id=12345)
vast_sdk.destroy_instance(id=12345)

# View instance details and logs
vast_sdk.show_instances()
vast_sdk.logs(id=12345)

# File operations
vast_sdk.copy(src="path", dst="path", identity="file")
vast_sdk.cloud_copy()

# SSH key management
vast_sdk.create_ssh_key()
vast_sdk.show_ssh_keys()
vast_sdk.delete_ssh_key(id=1)
```

### Serverless SDK

```python
import asyncio
from vastai import Serverless

async def main():
    client = Serverless()  # Uses VAST_API_KEY env var
    endpoint = await client.get_endpoint(name="vLLM-Qwen3-8B")

    payload = {
        "model": "Qwen/Qwen3-8B",
        "prompt": "Explain quantum computing in simple terms",
        "max_tokens": 100,
        "temperature": 0.7,
    }

    result = await endpoint.request("/v1/completions", payload, cost=100)
    print(result["response"]["choices"][0]["text"])
    await client.close()

asyncio.run(main())
```

## Configuration

### Instance Configuration

Instance parameters are specified at launch time through the CLI, SDK, or web console:

- **GPU Type**: Specific GPU model (e.g., RTX 3090, RTX 4090, A100, H100).
- **GPU Count**: Number of GPUs per instance.
- **RAM**: Minimum system RAM requirement.
- **CPU Cores**: Minimum CPU core count.
- **Disk Space**: Storage allocation (static, cannot be modified after creation).
- **Bandwidth**: Minimum network bandwidth.
- **Template/Image**: Docker image or template hash for the instance environment.
- **Launch Mode**: Entrypoint, SSH, or Jupyter.

### Environment Variables

Instances support custom environment variables via `-e` syntax. Predefined variables injected by the platform include: `CONTAINER_API_KEY`, `CONTAINER_ID`, `GPU_COUNT`, `PUBLIC_IPADDR`, `SSH_PUBLIC_KEY`, and port mapping variables (`VAST_TCP_PORT_X`, `VAST_UDP_PORT_X`). UI control variables include `OPEN_BUTTON_PORT`, `JUPYTER_PORT`, `JUPYTER_TOKEN`, and `DATA_DIRECTORY`.

### Serverless Endpoint Configuration

Endpoint-level parameters set during creation:

- **Endpoint Name**: Descriptive identifier for the endpoint.
- **Cold Multiplier**: Scales total capacity based on predicted load (e.g., 3x).
- **Minimum Workers**: Pre-loaded instances for instant scaling.
- **Maximum Workers**: Upper bound on GPU instances.
- **Minimum Load**: Baseline tokens-per-second instantaneous capacity.
- **Minimum Cold Load**: Baseline tokens-per-second total capacity.
- **Target Utilization**: Resource usage target (e.g., 0.9 for 90%).

Workergroup parameters include `gpu_ram` (VRAM in GB, default 24), `search_params` (hardware filtering criteria), `template_hash` or `template_id` (pre-configured deployment template), and `launch_args` (additional instance creation parameters).

### PyWorker Configuration

Custom PyWorker configuration in `worker.py`:

```python
from vastai import Worker, WorkerConfig, HandlerConfig, LogActionConfig, BenchmarkConfig

worker_config = WorkerConfig(
    model_server_url="http://127.0.0.1",
    model_server_port=18000,
    model_log_file="/var/log/portal/vllm.log",
    handlers=[
        HandlerConfig(
            route="/v1/completions",
            allow_parallel_requests=True,
            max_queue_time=60.0,
            workload_calculator=lambda p: float(p.get("max_tokens", 0)),
            benchmark_config=BenchmarkConfig(
                generator=completions_benchmark_generator,
                runs=8,
                concurrency=10,
            ),
        ),
    ],
    log_action_config=LogActionConfig(
        on_load=["Application startup complete."],
        on_error=["RuntimeError: Engine", "Traceback (most recent call last):"],
    ),
)

Worker(worker_config).run()
```

## Integration Patterns

### CLI Scripting

```bash
#!/bin/bash
# Find cheapest A100 offer and launch with PyTorch template
OFFER_ID=$(vastai search offers --gpu-name A100 --order dph | head -1 | awk '{print $1}')
vastai create instance $OFFER_ID --image pytorch/pytorch:latest --disk 64 --ssh --direct
```

### REST API Integration

```python
import requests

API_KEY = "your_api_key"
BASE_URL = "https://console.vast.ai/api/v0"
headers = {"Authorization": f"Bearer {API_KEY}"}

# Search for available offers
response = requests.get(f"{BASE_URL}/bundles/", headers=headers)
offers = response.json()
```

### Serverless Deployment via Git Repository

Deploy custom PyWorkers by creating a Git repository with `worker.py` and `requirements.txt`, then setting the `PYWORKER_REPO` environment variable in the serverless configuration. The platform clones the repository, installs dependencies, starts the model server, and runs the PyWorker.

### Cloud Storage Sync

```bash
# Copy data from S3 to a stopped instance
vastai cloud copy s3://bucket/data instance_id:/workspace/data

# Copy between instances (same datacenter avoids bandwidth charges)
vastai copy 12345:/output/ 67890:/input/
```

### SSH Automation

```bash
# Get SSH connection details
vastai ssh-url INSTANCE_ID

# SCP for smaller transfers (under 1 GB recommended)
scp -P PORT local_file.tar.gz root@IPADDR:/workspace/
```

## Examples

### Deploy a Serverless vLLM Endpoint

1. Navigate to the Serverless Dashboard at cloud.vast.ai/serverless.
2. Click "Get Started" and configure the endpoint with a name, cold multiplier of 3, minimum 5 workers, and maximum 16 workers.
3. Select the "vLLM (Serverless)" template pre-configured with Qwen/Qwen3-8B.
4. Click "Create" and wait 3-5 minutes for workers to initialize (download model, load into GPU memory, complete health checks).
5. Monitor worker states: Stopped (cold, ready to activate), Loading (starting), Ready (serving requests).

### Launch a Training Instance via CLI

```bash
# Search for A100 instances under $1/hr with high reliability
vastai search offers --gpu-name A100 --dph 1.0 --order dph 'reliability > 0.95'

# Create instance with 64 GB disk
vastai create instance 12345 --image pytorch/pytorch:2.0-cuda11.8-cudnn8-devel --disk 64 --ssh

# Monitor instance status
vastai show instances

# Stop instance (preserves data, stops GPU charges)
vastai stop instance 12345

# Destroy instance (stops all charges including storage)
vastai destroy instance 12345
```

### Reserved Instance for Long-Term Training

```bash
# Search for reservable H100 offers
vastai search offers --gpu-name H100 --reservable

# Create and convert to reserved for up to 50% savings
vastai create instance 67890 --image nvidia/cuda:12.0-devel --reserve
```

### Custom PyWorker for Image Generation

```python
from vastai import Worker, WorkerConfig, HandlerConfig, LogActionConfig, BenchmarkConfig

worker_config = WorkerConfig(
    model_server_url="http://127.0.0.1",
    model_server_port=8188,
    model_log_file="/var/log/portal/comfyui.log",
    handlers=[
        HandlerConfig(
            route="/api/generate",
            allow_parallel_requests=False,
            max_queue_time=120.0,
            workload_calculator=lambda p: 1.0,
        ),
    ],
    log_action_config=LogActionConfig(
        on_load=["ComfyUI startup complete"],
        on_error=["RuntimeError"],
    ),
)

Worker(worker_config).run()
```

## Limitations

- **Provider Variability**: Hardware reliability and network quality vary across providers. Reliability scores help mitigate risk, but interruptions can occur on lower-rated machines.
- **No Guaranteed Uptime SLA**: Unlike traditional cloud providers, Vast.ai does not offer enterprise-grade Service Level Agreements (SLAs) on most instances. Interruptible instances may be paused at any time by higher-priority rentals.
- **Data Locality**: Data must be transferred to and from instances. SCP over proxy SSH is recommended only for transfers under 1 GB; direct SSH connections or cloud sync are preferred for larger datasets.
- **Static Disk Allocation**: Disk space is set at instance creation and cannot be modified afterward.
- **Networking Constraints**: Instances share public IPs with port-based routing, limited to 64 ports per instance. No dedicated public IP assignment is available.
- **VM Limitations**: VM instances have slower boot times, higher disk overhead, limited machine availability, restricted preconfigured templates, and no SSH key modification on running instances. VM copy operations only support complete VM-to-VM transfers.
- **Closed Source Platform**: The marketplace platform is proprietary. The CLI and SDK are open source, but the core infrastructure is not.
- **Pre-Payment Required**: Credits must be purchased before launching instances. When balance reaches zero, instances are stopped automatically. Without a saved payment method, instances and stored data are destroyed.
- **Security Model**: Instances run on third-party hardware. Sensitive workloads require additional encryption and security measures beyond the platform's default Caddy proxy with TLS.

## Changelog

Vast.ai continuously updates its marketplace and tooling. Key platform capabilities as of the documentation crawl include serverless inference with benchmark-driven autoscaling, virtual machine instance support, the PyWorker custom worker framework, the Python SDK with async serverless client, cloud sync with S3 and Google Drive, team management with role-based access, and the Instance Portal web interface. Consult the official documentation and community Discord for the latest platform changes.

## Citations

- [1] Welcome to Vast.ai - [https://docs.vast.ai/documentation/get-started](https://docs.vast.ai/documentation/get-started)
- [2] Instances Overview - [https://docs.vast.ai/documentation/instances/overview](https://docs.vast.ai/documentation/instances/overview)
- [3] Pricing - [https://docs.vast.ai/documentation/instances/pricing](https://docs.vast.ai/documentation/instances/pricing)
- [4] Instance Types - [https://docs.vast.ai/documentation/instances/choosing/instance-types](https://docs.vast.ai/documentation/instances/choosing/instance-types)
- [5] Docker Execution Environment - [https://docs.vast.ai/documentation/instances/docker-environment](https://docs.vast.ai/documentation/instances/docker-environment)
- [6] Virtual Machines - [https://docs.vast.ai/documentation/instances/virtual-machines](https://docs.vast.ai/documentation/instances/virtual-machines)
- [7] Data Movement - [https://docs.vast.ai/documentation/instances/storage/data-movement](https://docs.vast.ai/documentation/instances/storage/data-movement)
- [8] Templates Introduction - [https://docs.vast.ai/documentation/templates/introduction](https://docs.vast.ai/documentation/templates/introduction)
- [9] Serverless Overview - [https://docs.vast.ai/documentation/serverless](https://docs.vast.ai/documentation/serverless)
- [10] Serverless Quickstart - [https://docs.vast.ai/documentation/serverless/quickstart](https://docs.vast.ai/documentation/serverless/quickstart)
- [11] Creating Custom PyWorkers - [https://docs.vast.ai/documentation/serverless/creating-new-pyworkers](https://docs.vast.ai/documentation/serverless/creating-new-pyworkers)
- [12] Workergroup Parameters - [https://docs.vast.ai/documentation/serverless/workergroup-parameters](https://docs.vast.ai/documentation/serverless/workergroup-parameters)
- [13] CLI Getting Started - [https://docs.vast.ai/cli/get-started](https://docs.vast.ai/cli/get-started)
- [14] Python SDK Quickstart - [https://docs.vast.ai/sdk/python/quickstart](https://docs.vast.ai/sdk/python/quickstart)
- [15] API Reference - [https://docs.vast.ai/api-reference/introduction](https://docs.vast.ai/api-reference/introduction)
- [16] Billing - [https://docs.vast.ai/documentation/reference/billing](https://docs.vast.ai/documentation/reference/billing)
- [17] Keys - [https://docs.vast.ai/documentation/reference/keys](https://docs.vast.ai/documentation/reference/keys)
