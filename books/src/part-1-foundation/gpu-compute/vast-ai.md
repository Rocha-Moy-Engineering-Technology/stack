# Vast.ai

| Field          | Value                                      |
|----------------|--------------------------------------------|
| **Group**      | GPU Compute & Cloud Platforms              |
| **Type**       | API/Infra                                  |
| **Open Source** | No                                        |
| **GitHub**     | N/A                                        |
| **Stars**      | N/A                                        |
| **Docs**       | [Official Docs](https://docs.vast.ai/)     |

## Overview

Vast.ai is a decentralized GPU marketplace that connects compute providers -- ranging from hobbyists to Tier-4 datacenters -- with users who need GPU resources for AI and machine learning workloads. The platform operates on a peer-to-peer model where providers maintain full pricing autonomy through dynamic pricing, and GPU instances can be launched in seconds. The stated mission is to democratize AI compute: "compute powering AI is supplied by the people and for the people."

## Core Concepts

- **GPU Marketplace**: A peer-to-peer compute network where providers list their hardware and set their own prices. Users browse available machines, compare specs and reliability ratings, and rent GPU time at market-driven rates.
- **Instances**: GPU instances with customizable specifications including GPU type, RAM, CPU cores, and bandwidth. Vast.ai offers both on-demand instances (pay as you go, no commitment) and reserved instances (longer-term commitments with up to 50% savings over on-demand pricing).
- **Templates**: Prebuilt environments that enable one-click deployment of common AI/ML frameworks and tools. Users can also create and share custom templates tailored to their specific workflows.
- **Search Engine**: A hardware search and filtering system that allows users to query available machines by GPU model, VRAM, CPU cores, RAM, disk space, bandwidth, provider reliability score, and price.

## Installation and Setup

Vast.ai provides a command-line interface (CLI) tool called `vastai` for programmatic interaction with the marketplace.

```bash
pip install vastai
vastai set api-key YOUR_API_KEY
```

After installation, verify the setup by listing available offers:

```bash
vastai search offers
```

API keys are generated from the Vast.ai web console and used for both CLI and REST API authentication.

## Architecture

Vast.ai follows a marketplace architecture with three main layers:

- **Provider Layer**: Individual GPU owners and datacenter operators register their machines on the platform, configure pricing, and make hardware available to the network.
- **Marketplace Layer**: The central platform handles instance discovery, search and filtering, transaction management, and reliability tracking. Dynamic pricing is determined by providers, not the platform.
- **Consumer Layer**: Users interact with the marketplace through the web console, CLI, or REST API to find, launch, and manage GPU instances.

Access to running instances is provided through multiple methods: SSH, Jupyter notebooks, and web portals.

## Key Features

- **Instance Search with Hardware Filtering**: Query available GPUs by model, VRAM, price, reliability, and other hardware specifications to find the best match for a given workload.
- **Dynamic Marketplace Pricing**: Providers set and adjust their own prices, creating a competitive market that typically offers lower rates than traditional cloud providers.
- **Reserved Instances**: Commit to longer-term rentals for up to 50% savings compared to on-demand pricing.
- **Data Transfer Tools**: Built-in utilities for uploading datasets to and downloading results from instances.
- **Instance Management**: Start, stop, restart, and monitor instances through the CLI, API, or web console.
- **Multiple Access Methods**: Connect to instances via SSH, Jupyter notebooks, or web-based portals depending on the workflow.
- **Template System**: Use prebuilt or custom templates for rapid environment setup and reproducible deployments.

## Use Cases

- **Model Training**: Rent high-end GPUs (A100, H100) for training large models at marketplace rates that are often significantly below major cloud providers.
- **Inference at Scale**: Deploy inference endpoints on cost-effective GPU instances with the ability to scale horizontally across multiple providers.
- **Experimentation and Prototyping**: Quickly spin up GPU instances for short-lived experiments without long-term commitments or upfront costs.
- **Batch Processing**: Run large batch jobs across multiple instances using on-demand pricing, paying only for the compute time consumed.
- **Fine-Tuning**: Use mid-range GPU instances to fine-tune pretrained models on custom datasets at competitive rates.

## API Reference Summary

Vast.ai exposes a REST API at the `/api/v0/` endpoint. Authentication uses Bearer tokens.

### Instance SSH Attachment

```
POST /api/v0/instances/{id}/ssh/
Authorization: Bearer <API_KEY>
```

Attaches an SSH key to a running instance for remote access.

### CLI Equivalents

```bash
# Attach SSH key to an instance
vastai attach ssh <instance_id> <ssh_key>

# Search for available offers
vastai search offers

# Create an instance
vastai create instance <offer_id> --image <template>

# Stop an instance
vastai stop instance <instance_id>

# Destroy an instance
vastai destroy instance <instance_id>
```

## Configuration

Instance configuration is specified at launch time through the CLI or API:

- **GPU Type**: Select specific GPU models (e.g., RTX 3090, A100, H100).
- **GPU Count**: Number of GPUs per instance.
- **RAM**: Minimum system RAM requirement.
- **CPU Cores**: Minimum CPU core count.
- **Disk Space**: Storage allocation for the instance.
- **Bandwidth**: Minimum network bandwidth.
- **Template/Image**: The Docker image or template to use for the instance environment.

## Integration Patterns

### CLI Scripting

```bash
#!/bin/bash
# Find cheapest A100 instance and launch with a PyTorch template
OFFER_ID=$(vastai search offers --gpu-name A100 --order dph | head -1 | awk '{print $1}')
vastai create instance $OFFER_ID --image pytorch/pytorch:latest
```

### REST API Integration

```python
import requests

API_KEY = "your_api_key"
BASE_URL = "https://console.vast.ai/api/v0"

headers = {"Authorization": f"Bearer {API_KEY}"}

# Search for offers
response = requests.get(f"{BASE_URL}/bundles/", headers=headers)
offers = response.json()
```

### SSH Automation

```bash
# Attach key and connect
vastai attach ssh <instance_id> ~/.ssh/id_rsa.pub
ssh -p <port> root@<host>
```

## Examples

### Launch a Training Instance

```bash
# Search for A100 instances under $1/hr
vastai search offers --gpu-name A100 --dph 1.0 --order dph

# Create instance from an offer
vastai create instance 12345 --image pytorch/pytorch:2.0-cuda11.8-cudnn8-devel

# Monitor instance status
vastai show instances
```

### Reserved Instance Workflow

```bash
# Search for reservable offers
vastai search offers --gpu-name H100 --reservable

# Create a reserved instance for cost savings
vastai create instance 67890 --image nvidia/cuda:12.0-devel --reserve
```

## Limitations

- **Provider Variability**: Hardware reliability and network quality vary across providers. Reliability scores help mitigate this, but interruptions can occur on lower-rated machines.
- **No Guaranteed Uptime SLA**: Unlike traditional cloud providers, Vast.ai does not offer enterprise-grade Service Level Agreements (SLAs) on most instances.
- **Data Locality**: Data must be transferred to and from instances, which can introduce latency for large datasets.
- **Security Considerations**: Instances run on third-party hardware. Sensitive workloads may require additional encryption and security measures.
- **Closed Source Platform**: The marketplace platform itself is proprietary and not open source.
- **Spot-Like Behavior**: On-demand instances on shared hardware may be preempted if a higher-priority reservation is placed by the provider.

## Changelog Highlights

Vast.ai continuously updates its marketplace and tooling. Consult the official documentation and announcements for the latest platform changes, new GPU availability, and CLI/API updates.

## Citations

- [1] Vast.ai Documentation - [https://docs.vast.ai/](https://docs.vast.ai/)
