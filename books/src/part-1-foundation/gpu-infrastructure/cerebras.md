# Cerebras

> AI inference API on custom wafer-scale engine hardware

| Field | Value |
|-------|-------|
| Group | GPU Compute & Cloud Platforms |
| Type | API/Infra |
| Open Source | No |
| GitHub | N/A |
| Stars | N/A |
| Documentation | [Official Docs](https://inference-docs.cerebras.ai/) |

## Overview

Cerebras is an AI inference platform built on custom wafer-scale engine hardware (CS-3 chips). Rather than relying on traditional GPU clusters, Cerebras uses its proprietary chip architecture to deliver high-throughput, low-latency Large Language Model (LLM) inference. The platform exposes a cloud API that is OpenAI-compatible, allowing developers to switch from OpenAI endpoints with minimal code changes. Cerebras targets use cases where inference speed is critical, serving both general-purpose chat completions and advanced capabilities such as reasoning models, structured outputs, and function calling.

## Core Concepts

- **Wafer-Scale Engine (WSE):** Cerebras hardware is based on wafer-scale chips (CS-3), where an entire silicon wafer acts as a single processor. This eliminates the inter-chip communication bottleneck found in traditional GPU clusters, enabling faster sequential token generation.
- **OpenAI-Compatible API:** The chat completions endpoint follows the same request and response schema as the OpenAI API, making migration straightforward for applications already built against that interface.
- **Service Tiers:** Cerebras offers configurable service tiers (priority, default, auto, flex) that control request scheduling and queue behavior, allowing users to trade off between latency guarantees and cost.
- **Reasoning Effort:** For supported models (such as gpt-oss-120b), a `reasoning_effort` parameter (low, medium, high) controls how much compute the model allocates to chain-of-thought reasoning before producing a final answer.
- **Prompt Caching:** Repeated prompt prefixes can be cached server-side, reducing time-to-first-token for workloads that share common system prompts or context across requests.

## Installation and Setup

### Python SDK

Install the Python SDK via pip:

```bash
pip install cerebras-cloud-sdk
```

Set the API key as an environment variable:

```bash
export CEREBRAS_API_KEY="your_api_key_here"
```

### JavaScript SDK

Install the JavaScript SDK via npm:

```bash
npm install @cerebras/cerebras_cloud_sdk
```

### REST / curl

No SDK installation is required. Authenticate by passing the API key as a Bearer token in the Authorization header.

```bash
curl -X POST https://api.cerebras.ai/v1/chat/completions \
  -H "Authorization: Bearer $CEREBRAS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-oss-120b",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

## Architecture

Cerebras operates as a managed cloud inference service. The architecture has three layers:

1. **Client Layer:** Applications interact with the platform through the Python SDK (`cerebras.cloud.sdk`), the JavaScript SDK (`@cerebras/cerebras_cloud_sdk`), or direct REST calls against the `https://api.cerebras.ai/v1/` base URL.
2. **API Gateway:** Handles authentication (Bearer token), request validation, service tier routing, and queue management. The gateway exposes an OpenAI-compatible interface so that existing tooling and libraries designed for OpenAI work without modification.
3. **Inference Engine:** Requests are dispatched to CS-3 wafer-scale engine hardware for model execution. The hardware architecture eliminates multi-chip communication overhead, producing low-latency token generation. Prompt caching is handled at this layer, reusing previously computed key-value states for shared prompt prefixes.

The response includes detailed `time_info` metrics (`queue_time`, `prompt_time`, `completion_time`, `total_time`) that expose the performance characteristics of each layer.

## Key Features and Functionality

- **High-Speed Inference:** Wafer-scale hardware delivers fast token generation compared to traditional GPU-based inference platforms, particularly for long-context and high-throughput workloads.
- **Prompt Caching:** Server-side caching of prompt prefixes reduces redundant computation when multiple requests share common context (such as system prompts or few-shot examples).
- **Structured Outputs:** The `response_format` parameter supports `json_object` and `json_schema` modes, enforcing that model output conforms to a user-defined JSON schema.
- **Streaming:** Real-time token streaming via Server-Sent Events (SSE) when the `stream` parameter is set to true, enabling progressive rendering in user interfaces.
- **Tool Use and Function Calling:** The `tools` parameter accepts function definitions that the model can invoke. Parallel tool calls are supported via `parallel_tool_calls`, allowing the model to request multiple function executions in a single response turn.
- **Reasoning Models:** The gpt-oss-120b model supports configurable reasoning effort (low, medium, high) through the `reasoning_effort` parameter, controlling the depth of chain-of-thought processing.
- **Predicted Outputs:** The `prediction` parameter allows clients to supply an expected output, enabling the engine to accelerate generation when the prediction is close to the actual response.
- **Clear Thinking:** The zai-glm-4.7 model supports a `clear_thinking` parameter for controlling the visibility of internal reasoning steps.

## Use Cases

- **Low-Latency Chat Applications:** The speed of wafer-scale inference makes Cerebras suitable for real-time conversational interfaces where response latency directly affects user experience.
- **Batch Inference Pipelines:** The flex service tier and prompt caching enable cost-effective processing of large volumes of requests that share common prefixes.
- **Structured Data Extraction:** JSON schema-enforced outputs are useful for extracting structured information from unstructured text (such as entity extraction, form parsing, or data normalization).
- **Agentic Tool Use:** Function calling with parallel execution supports agentic workflows where the model orchestrates multiple external tool invocations per turn.
- **Reasoning-Heavy Tasks:** The configurable reasoning effort on gpt-oss-120b allows tuning the cost-accuracy tradeoff for tasks that benefit from extended chain-of-thought (such as math, coding, and multi-step analysis).

## API Reference Summary

### Endpoint

```
POST https://api.cerebras.ai/v1/chat/completions
```

### Authentication

```
Authorization: Bearer <CEREBRAS_API_KEY>
```

### Available Models

| Model | Notes |
|-------|-------|
| llama3.1-8b | General use |
| qwen-3-235b-a22b-instruct-2507 | Preview |
| gpt-oss-120b | Reasoning |
| zai-glm-4.7 | Preview |

### Request Parameters

**Required:**

- `model` (string): The model identifier.
- `messages` (array): An array of message objects, each with `role` (system, user, assistant) and `content` (string).

**Response Control:**

- `max_completion_tokens` (integer): Maximum number of tokens to generate.
- `temperature` (float, 0 to 1.5): Sampling temperature.
- `top_p` (float): Nucleus sampling threshold.
- `stream` (boolean): Enable streaming responses.
- `stop` (string or array): Up to 4 stop sequences.

**Advanced:**

- `reasoning_effort` (string: low, medium, high): Chain-of-thought depth for gpt-oss-120b.
- `response_format` (object): One of `text`, `json_object`, or `json_schema` with a schema definition.
- `tools` (array): Function definitions for tool use.
- `tool_choice` (string or object): Control which tools the model may call.
- `parallel_tool_calls` (boolean): Allow multiple tool calls in a single response.
- `seed` (integer): Deterministic sampling seed.
- `logprobs` (boolean): Return log probabilities of output tokens.
- `top_logprobs` (integer): Number of top log probabilities to return per token.
- `prediction` (object): Predicted output for accelerated generation.
- `clear_thinking` (boolean): Control reasoning visibility for zai-glm-4.7.
- `user` (string): End-user identifier for tracking.

**Service:**

- `service_tier` (string: priority, default, auto, flex): Request scheduling tier.
- `queue_threshold` (integer): Maximum acceptable queue time before rejection.

### Response Structure

- `choices` (array): Array of completion choices, each containing `message` (with `role` and `content`), `finish_reason`, and optionally `tool_calls`.
- `usage` (object): Token usage metrics (`prompt_tokens`, `completion_tokens`, `total_tokens`).
- `time_info` (object): Timing breakdown with `queue_time`, `prompt_time`, `completion_time`, and `total_time`.

## Configuration and Customization

### Environment Variables

| Variable | Description |
|----------|-------------|
| `CEREBRAS_API_KEY` | API key for authentication |

### Service Tier Selection

- **priority:** Lowest latency, highest cost. Requests are processed immediately.
- **default:** Standard scheduling with balanced latency and cost.
- **auto:** Platform selects the optimal tier based on current load.
- **flex:** Lowest cost, requests may be queued during peak periods. Suitable for batch workloads.

The `queue_threshold` parameter (in seconds) can be combined with any service tier to reject requests that would wait longer than the specified threshold.

## Integration Patterns

### Drop-In Replacement for OpenAI

Because the API is OpenAI-compatible, applications using the OpenAI Python SDK can switch to Cerebras by changing the base URL and API key:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.cerebras.ai/v1",
    api_key="your_cerebras_api_key"
)

response = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[{"role": "user", "content": "Explain wafer-scale computing."}]
)
```

### Framework Compatibility

Any framework or library that supports OpenAI-compatible endpoints (such as LangChain, LlamaIndex, or custom orchestrators) can be pointed at the Cerebras API by configuring the base URL.

## Examples

### Basic Chat Completion (Python SDK)

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

chat_completion = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[{"role": "user", "content": "Hello!"}]
)

print(chat_completion.choices[0].message.content)
```

### Streaming Response

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

stream = client.chat.completions.create(
    model="llama3.1-8b",
    messages=[{"role": "user", "content": "Write a short poem."}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Structured Output with JSON Schema

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

response = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[
        {"role": "user", "content": "Extract the name and age from: John is 30 years old."}
    ],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "person_info",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "age": {"type": "integer"}
                },
                "required": ["name", "age"]
            }
        }
    }
)

print(response.choices[0].message.content)
```

### Function Calling

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name"
                    }
                },
                "required": ["location"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[{"role": "user", "content": "What is the weather in San Francisco?"}],
    tools=tools,
    tool_choice="auto"
)

print(response.choices[0].message.tool_calls)
```

### Reasoning with Configurable Effort

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your_key")

response = client.chat.completions.create(
    model="gpt-oss-120b",
    messages=[
        {"role": "user", "content": "Prove that the square root of 2 is irrational."}
    ],
    reasoning_effort="high"
)

print(response.choices[0].message.content)
```

## Limitations and Considerations

- **Closed Source Hardware and Software:** The wafer-scale engine and inference runtime are proprietary. Users cannot self-host or inspect the inference pipeline.
- **Model Selection:** The available model catalog is limited compared to general-purpose GPU cloud platforms. Only a small set of models are supported at any given time, and some are in preview status.
- **No Fine-Tuning:** The platform provides inference only. Users cannot fine-tune or train custom models on Cerebras hardware through the cloud API.
- **Regional Availability:** As a specialized hardware platform, availability may be constrained by data center locations and capacity.
- **Preview Model Stability:** Models marked as preview (qwen-3-235b-a22b-instruct-2507, zai-glm-4.7) may change behavior or be removed without notice.
- **Parameter Restrictions:** Some advanced parameters are model-specific (`reasoning_effort` works only with gpt-oss-120b; `clear_thinking` works only with zai-glm-4.7), requiring conditional logic when supporting multiple models.

## Changelog Highlights

Cerebras does not publish a public changelog for the inference API. Model availability and parameter support should be verified against the current documentation, as the platform is actively evolving.

## Citations

- [1] Cerebras Inference Documentation - [https://inference-docs.cerebras.ai/](https://inference-docs.cerebras.ai/)
- [2] Cerebras API Reference - [https://inference-docs.cerebras.ai/api-reference/chat-completions](https://inference-docs.cerebras.ai/api-reference/chat-completions)
