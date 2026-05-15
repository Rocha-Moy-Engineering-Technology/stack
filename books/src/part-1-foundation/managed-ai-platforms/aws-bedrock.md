# AWS Bedrock

> AWS managed service for foundation model access from multiple vendors via single API

| Field | Value |
|-------|-------|
| Group | Managed AI Platforms |
| Type | API/SDK/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://docs.aws.amazon.com/bedrock/) |

## Overview

Amazon Bedrock is a fully managed service that provides secure, enterprise-grade access to over 100 foundation models from leading AI companies through a single API. It enables organizations to build and scale generative AI applications without managing infrastructure, offering model inference, customization, retrieval-augmented generation (RAG), agentic workflows, and safety guardrails as integrated capabilities.

Bedrock abstracts away the complexity of hosting and serving large language models. Users select from a catalog of models spanning multiple providers -- including Amazon, Anthropic, OpenAI, DeepSeek, Moonshot AI, MiniMax, GLM, Qwen, and NVIDIA -- and interact with them through unified APIs. The service handles provisioning, scaling, and security, allowing teams to focus on application logic rather than infrastructure management.

## Core Concepts

- **Foundation Models**: Pre-trained large language models available through the Bedrock model catalog, spanning text generation, code generation, and multimodal tasks.
- **Model Inference**: Submitting prompts to a selected model and receiving generated responses via API calls.
- **Model Customization**: Fine-tuning foundation models on proprietary data to improve performance for domain-specific tasks.
- **Knowledge Bases**: Managed RAG infrastructure that integrates proprietary data sources so models can ground responses in organizational knowledge.
- **Agents**: Agentic application framework enabling models to plan, invoke tools, and execute multi-step workflows autonomously.
- **Guardrails**: Configurable safety and compliance controls that filter model inputs and outputs according to organizational policies.
- **Prompt Caching**: Server-side caching mechanism (1-hour duration for Claude models) that reduces latency and cost for repeated or similar prompts.

## Architecture

Bedrock operates as a managed API layer between client applications and foundation model infrastructure. The architecture consists of three tiers:

1. **Client Tier**: Applications interact with Bedrock through the `bedrock-runtime` API endpoint using boto3, the OpenAI-compatible endpoint, or the AWS CLI.
2. **Service Tier**: Bedrock routes requests to the appropriate model provider, applies guardrails, manages prompt caching, and orchestrates agent workflows.
3. **Model Tier**: Foundation models from multiple providers run on AWS-managed GPU infrastructure, isolated per customer with no cross-tenant data exposure.

All data remains within the customer's AWS account and selected region. Model providers do not receive access to customer data, and customer inputs are not used to train or improve third-party models.

## Key Features and Functionality

- **100+ Foundation Models**: Single API access to models from Amazon (Nova 2 Pro), Anthropic (Claude Opus 4.6), OpenAI (GPT-OSS-20B), DeepSeek (V3.2), Moonshot AI (Kimi K2.5), MiniMax (M2.1), GLM (4.7, 4.7 Flash), Qwen (3 Coder Next), and NVIDIA (Nemotron 3 Nano).
- **Multiple API Formats**: Converse API (recommended), Invoke API (Bedrock-native), Chat Completions API (OpenAI-compatible), and Responses API (OpenAI-compatible).
- **Knowledge Bases for RAG**: Managed vector storage and retrieval pipeline for grounding model responses in proprietary documents.
- **Agentic Applications**: Built-in agent framework with tool calling, multi-step planning, and external system integration.
- **Guardrails**: Content filtering, topic restrictions, personally identifiable information (PII) redaction, and custom policy enforcement.
- **Server-Side Tools**: Tool execution via OpenAI-compatible endpoints, reducing client-side orchestration complexity.
- **Prompt Caching**: 1-hour server-side cache for Claude models, lowering latency and cost on repeated prompt patterns.
- **NVIDIA Nemotron 3 Nano**: 256k context window with native tool calling support.
- **Open-Weight Models**: Six open-weight models available (DeepSeek V3.2, MiniMax M2.1, GLM 4.7, GLM 4.7 Flash, Kimi K2.5, Qwen3 Coder Next).
- **Model Customization**: Fine-tuning and continued pre-training on proprietary datasets.

## Use Cases

- **Enterprise Chat and Assistants**: Deploy conversational AI powered by best-in-class models with organizational data grounding through Knowledge Bases.
- **Code Generation and Review**: Use code-specialized models (Qwen3 Coder Next, Claude Opus 4.6) for automated code generation, explanation, and review.
- **Document Processing**: Extract, summarize, and transform information from large document corpora using long-context models.
- **Agentic Workflows**: Build autonomous agents that plan multi-step tasks, invoke external APIs, query databases, and produce structured outputs.
- **Content Moderation**: Apply Guardrails to filter harmful or non-compliant content in user-facing applications.
- **RAG Applications**: Combine Knowledge Bases with foundation models to answer questions grounded in proprietary enterprise data.

## API Reference Summary

### Converse API (Recommended)

The Converse API provides a unified interface across all Bedrock models with consistent request and response formats.

```python
import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.converse(
    modelId='anthropic.claude-opus-4-6-v1',
    messages=[
        {
            'role': 'user',
            'content': [{'text': 'Hello'}]
        }
    ]
)
print(response['output']['message']['content'][0]['text'])
```

### Invoke API (Bedrock-Native)

The Invoke API provides direct model invocation with provider-specific request body formats.

```python
import json
import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.invoke_model(
    modelId='anthropic.claude-opus-4-6-v1',
    body=json.dumps({
        'anthropic_version': 'bedrock-2023-05-31',
        'messages': [{'role': 'user', 'content': 'Hello'}],
        'max_tokens': 1024
    })
)
result = json.loads(response['body'].read())
print(result['content'][0]['text'])
```

### Chat Completions API (OpenAI-Compatible)

The OpenAI-compatible endpoint allows existing OpenAI SDK integrations to target Bedrock models with minimal code changes.

```python
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.create(
    model="openai.gpt-oss-120b",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

### Responses API (OpenAI-Compatible)

The Responses API provides an OpenAI-compatible endpoint supporting server-side tool execution and structured responses.

## Configuration and Customization

### IAM Permissions

Bedrock requires IAM policies granting access to specific actions and model resources:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "bedrock:Converse",
                "bedrock:ListFoundationModels"
            ],
            "Resource": "arn:aws:bedrock:us-east-1::foundation-model/*"
        }
    ]
}
```

### Region Selection

Bedrock availability and model catalogs vary by AWS region. US East (N. Virginia, `us-east-1`) and US West (Oregon, `us-west-2`) offer the broadest model selection.

### Model Access

Foundation models must be explicitly enabled in the Bedrock console before they can be invoked via API. Navigate to **Model access** in the Bedrock console and request access to desired models.

## Integration Patterns

### Direct API Integration

The simplest pattern: application code calls Bedrock APIs directly using boto3 or the OpenAI-compatible endpoint.

```python
import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.converse(
    modelId='amazon.nova-2-pro-v1',
    messages=[{'role': 'user', 'content': [{'text': 'Summarize this document.'}]}]
)
```

### RAG with Knowledge Bases

Combine document ingestion, vector search, and model inference through Bedrock Knowledge Bases:

```python
import boto3

agent_client = boto3.client('bedrock-agent-runtime', region_name='us-east-1')
response = agent_client.retrieve_and_generate(
    input={'text': 'What is our refund policy?'},
    retrieveAndGenerateConfiguration={
        'type': 'KNOWLEDGE_BASE',
        'knowledgeBaseConfiguration': {
            'knowledgeBaseId': 'YOUR_KB_ID',
            'modelArn': 'arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-opus-4-6-v1'
        }
    }
)
```

### OpenAI SDK Migration

Existing applications using the OpenAI SDK can target Bedrock by configuring the base URL and authentication:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://bedrock-runtime.us-east-1.amazonaws.com/openai/v1",
)
```

## Examples

### Multi-Turn Conversation

```python
import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')

messages = []
messages.append({'role': 'user', 'content': [{'text': 'What is machine learning?'}]})

response = client.converse(
    modelId='anthropic.claude-opus-4-6-v1',
    messages=messages
)

assistant_message = response['output']['message']
messages.append(assistant_message)

messages.append({'role': 'user', 'content': [{'text': 'Give me a concrete example.'}]})

response = client.converse(
    modelId='anthropic.claude-opus-4-6-v1',
    messages=messages
)
print(response['output']['message']['content'][0]['text'])
```

### Streaming Response

```python
import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.converse_stream(
    modelId='anthropic.claude-opus-4-6-v1',
    messages=[{'role': 'user', 'content': [{'text': 'Write a short essay on AI safety.'}]}]
)

for event in response['stream']:
    if 'contentBlockDelta' in event:
        print(event['contentBlockDelta']['delta']['text'], end='')
```

### System Prompt with Temperature

```python
import boto3

client = boto3.client('bedrock-runtime', region_name='us-east-1')
response = client.converse(
    modelId='anthropic.claude-opus-4-6-v1',
    system=[{'text': 'You are a helpful coding assistant. Respond with code examples.'}],
    messages=[{'role': 'user', 'content': [{'text': 'Write a Python function to parse CSV files.'}]}],
    inferenceConfig={'temperature': 0.2, 'maxTokens': 2048}
)
print(response['output']['message']['content'][0]['text'])
```

## Limitations and Considerations

- **Regional Availability**: Not all models are available in every AWS region. Model catalogs vary by region.
- **Model Access Approval**: Foundation models require explicit access requests before use; approval is not instant for all models.
- **Vendor Lock-In**: While the OpenAI-compatible API reduces switching cost, Knowledge Bases, Agents, and Guardrails are AWS-specific services.
- **Prompt Caching Scope**: 1-hour caching is currently limited to Claude models; other providers may have different or no caching behavior.
- **Rate Limits**: Throughput quotas are per-account and per-model; high-volume applications may require provisioned throughput.
- **Customization Constraints**: Fine-tuning is available only for a subset of models, not the full catalog.
- **Closed Source**: The service itself is proprietary; no self-hosting option exists.

## Changelog Highlights

- **2025 Q4**: Added six open-weight models (DeepSeek V3.2, MiniMax M2.1, GLM 4.7, GLM 4.7 Flash, Kimi K2.5, Qwen3 Coder Next).
- **2025 Q3**: NVIDIA Nemotron 3 Nano added with 256k context and native tool calling.
- **2025 Q2**: OpenAI-compatible Chat Completions and Responses APIs launched.
- **2025 Q1**: Amazon Nova 2 Pro model family released.
- **2024**: Guardrails general availability, Knowledge Bases general availability, Agents general availability, prompt caching for Claude models.

## Citations

- [1] AWS Bedrock Documentation - https://docs.aws.amazon.com/bedrock/
- [2] AWS Bedrock User Guide - https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- aws bedrock
- amazon bedrock
- bedrock-runtime
- foundation models
- model catalog
- converse api
- invoke_model
- chat completions api
- responses api
- knowledge bases
- bedrock agents
- guardrails
- prompt caching
- claude on bedrock
- amazon nova
- deepseek on bedrock
- nemotron
- kimi on bedrock
- glm on bedrock
- qwen on bedrock
- minimax on bedrock
- bedrock fine-tuning
- provisioned throughput
- iam bedrock policy
- region us-east-1
- bedrock-agent-runtime
- retrieve_and_generate
- open-weight models on aws

### Verb-Noun Tasks

- Call `converse()` to chat across multiple model providers
- Use `invoke_model` for provider-native request bodies
- Migrate an OpenAI SDK app to Bedrock by swapping the base URL
- Stream responses with `converse_stream`
- Build a RAG pipeline using Bedrock Knowledge Bases
- Wire up multi-step Bedrock Agents with tool calling
- Apply Guardrails to filter inputs and outputs
- Enable prompt caching for repeated Claude prompt prefixes
- Request model access in the Bedrock console
- Fine-tune a foundation model on proprietary data
- Configure IAM policies for `bedrock:InvokeModel` and `bedrock:Converse`
- Run `retrieve_and_generate` for grounded RAG answers

### User Intent Phrases

- How do I call Claude or DeepSeek through AWS?
- I want one API for 100+ foundation models without managing GPUs.
- How do I build RAG on AWS with managed vector storage?
- How do I configure content filters on LLM inputs and outputs?
- How do I fine-tune a foundation model on private data within my AWS account?
- How do I get prompt caching with Claude on AWS?
- How do I migrate from the OpenAI SDK to Bedrock with minimal code changes?
- How do I build a multi-step agent on AWS that calls my internal APIs?
- How do I grant IAM access to specific Bedrock models?
- Where do I request model access for Anthropic on Bedrock?

### Problem Statements

- Our data, identity, and billing are on AWS and we need LLM access without leaving the account boundary.
- We need regulated workloads where customer data never leaves our AWS region.
- We need managed RAG with no operational vector-database work.
- We want a unified API surface across providers without managing keys per vendor.
- We need content moderation as a built-in product, not a third-party integration.
- We need provisioned throughput for predictable inference latency at scale.

### When to Pick This

- Pick this when your data, identity, and billing are already on AWS (vs Vertex AI when you are on GCP).
- Pick this when you need managed Knowledge Bases and Agents as first-class AWS services (vs assembling RAG yourself on Modal/RunPod).
- Pick this when Guardrails, PII redaction, and topic restriction must run inside the same managed surface.
- Pick this when access to Anthropic Claude alongside Amazon Nova, DeepSeek, Kimi, GLM, MiniMax, Qwen, and Nemotron through one API matters.
- Pick this when an OpenAI-compatible endpoint reduces migration cost from existing OpenAI apps.
- Pick this when provisioned throughput and per-account model isolation are required for compliance.
- Pick this when fine-tuning on a subset of models inside the same AWS account is part of the workflow.

### Related Terms and Aliases

- aws gen ai
- managed foundation models
- aws llm api
- bedrock knowledge base
- bedrock guardrails
- bedrock agents
- aws ai platform
- enterprise llm gateway aws
- amazon nova
- aws prompt caching
