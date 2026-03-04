# RunPod

> GPU cloud platform offering pod instances and serverless GPU endpoints

| Field | Value |
|-------|-------|
| Group | GPU Compute & Cloud Platforms |
| Type | API/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.runpod.io/) |

## Overview

RunPod is a cloud computing platform purpose-built for AI, machine learning (ML), and general compute workloads. It provides scalable GPU and CPU resources for training, fine-tuning, and inference, with pricing models that range from pay-per-second serverless execution to dedicated per-minute pod instances. The platform emphasizes rapid deployment through preconfigured workers, container-based workflows, and managed multi-node clusters for distributed training.

## Core Concepts

- **Serverless Endpoints**: Access points that handle request routing and queuing, scaling worker instances automatically with zero idle costs and pay-per-second billing
- **Workers**: Container instances that execute compute tasks, managed by the platform with automatic scaling and lifecycle control
- **Handler Functions**: User-defined processing logic that receives input events and returns results, forming the core unit of serverless execution
- **Pods**: Dedicated GPU or CPU instances running containerized workloads, billed by the minute, suitable for long-running or interactive sessions
- **Public Endpoints**: Pre-deployed API access points to AI models covering image, video, audio, and text generation without requiring custom deployment
- **Instant Clusters**: Managed multi-node clusters with high-speed networking up to 3200 Gbps, supporting Slurm High-Performance Computing (HPC) scheduling and distributed PyTorch training
- **FlashBoot**: A cold start optimization mechanism that reduces worker startup latency through cached container images and model weights

## Installation and Setup

RunPod is a managed cloud platform and does not require local installation. Access is provisioned through the RunPod web console, CLI, or API.

1. Create an account at [runpod.io](https://www.runpod.io/)
2. Add billing credentials and select a compute plan
3. For serverless workloads, create an endpoint and deploy a worker container
4. For pod-based workloads, launch a pod instance with the desired GPU configuration
5. Optionally install the RunPod Python SDK for local development and testing:

```bash
pip install runpod
```

## Architecture

RunPod's serverless architecture follows a request-driven execution model:

1. **Request Arrival**: An HTTP request reaches the serverless endpoint
2. **Worker Startup**: If no active workers are available, a cold start initializes a new container instance
3. **Queuing**: Requests are queued when all active workers are occupied
4. **Handler Execution**: The handler function processes the input payload from the event object
5. **Result Return**: The computed result is returned to the caller
6. **Worker Persistence**: The worker remains active for a configurable idle period to serve subsequent requests
7. **Auto-Shutdown**: After the idle timeout expires, the worker shuts down to eliminate costs

Endpoint types serve different routing strategies:

- **Queue-Based Endpoints**: Sequential processing via `/run` (asynchronous) and `/runsync` (synchronous) routes, suited for batch and throughput-oriented workloads
- **Load Balancing Endpoints**: Direct routing to workers for real-time applications, compatible with FastAPI and Flask web frameworks

## Key Features and Functionality

- **Pay-Per-Second Billing**: Serverless compute charges accumulate only during active execution, with no cost for idle time
- **Automatic Scaling**: Worker instances scale up and down based on request volume without manual intervention
- **Cold Start Mitigation**: Multiple strategies reduce startup latency including cached models, FlashBoot prewarming, and configurable minimum active workers
- **Container-Based Deployment**: All workloads run in Docker containers, enabling reproducible environments across development and production
- **Managed Clustering**: Instant Clusters provide multi-node GPU environments with high-bandwidth interconnects for distributed training at scale
- **Rapid Deployment**: Fork existing workers from GitHub repositories, deploy vLLM workers for Large Language Model (LLM) serving, or use RunPod Hub preconfigured endpoints
- **Public Endpoints**: Instant API access to popular AI models without deploying custom infrastructure

## Use Cases

- **Model Inference**: Deploy trained models as serverless endpoints for on-demand prediction serving with automatic scaling
- **Model Training**: Use dedicated pods or instant clusters with multi-GPU configurations for training runs
- **Fine-Tuning**: Run parameter-efficient fine-tuning jobs on serverless infrastructure with pay-per-second billing
- **LLM Serving**: Deploy large language models using vLLM workers for high-throughput text generation
- **Batch Processing**: Queue large volumes of inference requests through queue-based endpoints for asynchronous processing
- **Distributed Training**: Leverage instant clusters with Slurm scheduling and distributed PyTorch for multi-node training across high-bandwidth GPU interconnects
- **Prototyping**: Use public endpoints for rapid experimentation with image, video, audio, and text generation models

## API Reference Summary

### Serverless Endpoints

- `POST /run` - Submit an asynchronous job to the queue, returns a job ID for polling
- `POST /runsync` - Submit a synchronous job that blocks until completion and returns the result directly
- `GET /status/{job_id}` - Retrieve the status and result of a previously submitted asynchronous job

### Handler Function Interface

The handler function is the entry point for serverless worker logic. It receives an event dictionary containing the input payload and returns the processed result:

```python
import runpod

def handler(event):
    input_data = event["input"]
    result = process_data(input_data)
    return result

runpod.serverless.start({"handler": handler})
```

## Configuration and Customization

- **Active Workers**: Set a minimum number of always-running workers to eliminate cold starts for latency-sensitive workloads
- **Max Workers**: Define the upper bound on concurrent worker instances to control cost
- **Idle Timeout**: Configure how long a worker remains active after completing its last request before auto-shutdown
- **GPU Selection**: Choose GPU type and quantity per worker or pod, with options varying by availability and region
- **Container Image**: Specify the Docker image and registry for worker deployment
- **Environment Variables**: Pass configuration values to workers through the endpoint or pod settings

## Integration Patterns

### Direct API Integration

Call RunPod serverless endpoints from any HTTP client:

```python
import requests

response = requests.post(
    "https://api.runpod.ai/v2/{endpoint_id}/runsync",
    headers={"Authorization": "Bearer {api_key}"},
    json={"input": {"prompt": "Hello, world"}}
)
result = response.json()
```

### vLLM Worker Deployment

Deploy a vLLM-based LLM serving worker by forking the RunPod vLLM worker template from GitHub and configuring the model path and serving parameters.

### Public Endpoint Consumption

Access pre-deployed models through public endpoint APIs for image generation, text generation, audio processing, and video generation without custom deployment.

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

### Asynchronous Job Submission

```python
import requests
import time

endpoint_url = "https://api.runpod.ai/v2/{endpoint_id}"
headers = {"Authorization": "Bearer {api_key}"}

# Submit async job
job = requests.post(
    f"{endpoint_url}/run",
    headers=headers,
    json={"input": {"prompt": "Generate an image"}}
).json()

# Poll for result
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

### Development Workflow

The recommended development lifecycle for serverless workers:

1. Write the handler function with processing logic
2. Test locally using the RunPod SDK test utilities
3. Package the handler into a Docker container
4. Push the container image to a registry (Docker Hub, GitHub Container Registry)
5. Deploy the image as a serverless endpoint through the RunPod console or API
6. Monitor endpoint metrics and logs
7. Iterate on the handler and redeploy

## Limitations and Considerations

- **Cold Start Latency**: Initial requests to idle endpoints incur startup delay as containers and models load; mitigated but not eliminated by FlashBoot and minimum active workers
- **Closed Source**: The platform itself is proprietary; users cannot self-host or inspect the orchestration layer
- **GPU Availability**: Specific GPU types may have limited availability depending on demand and region
- **Vendor Lock-In**: Serverless handler functions use the RunPod SDK interface, requiring adaptation to migrate to other platforms
- **Networking Constraints**: Inter-pod and cross-cluster networking is managed by the platform with limited user control over topology

## Changelog Highlights

RunPod evolves as a managed platform with continuous updates. Notable capabilities include the introduction of Instant Clusters for distributed training, FlashBoot for cold start optimization, Public Endpoints for zero-deployment model access, and vLLM worker templates for streamlined LLM serving. Consult the official documentation for the latest platform changes.

## Citations

- [1] RunPod Documentation - https://docs.runpod.io/
- [2] RunPod Serverless - https://docs.runpod.io/serverless/overview
