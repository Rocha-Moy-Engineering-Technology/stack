# Portkey

> AI gateway for managing 200+ LLMs with observability and guardrails

| Field | Value |
|-------|-------|
| Group | Model Gateways |
| Type | API/SDK |
| Open Source | Yes |
| GitHub | [Portkey-AI/gateway](https://github.com/Portkey-AI/gateway) |
| Stars | 11719 |
| Documentation | [Official Docs](https://portkey.ai/docs/introduction/what-is-portkey) |

## Overview

Portkey is a unified AI gateway that provides a single interface for routing requests to 250+ Large Language Model (LLM) providers. It operates as a proxy layer between applications and LLM APIs, adding observability, guardrails, caching, fallback routing, and load balancing without requiring changes to the underlying model calls. The gateway runs on globally distributed edge workers, adding approximately 20-40ms of latency compared to direct API calls. [1]

The platform processes over 25 million requests daily with 99.99% uptime and handles millions of requests per minute at scale. Integration takes approximately 2 minutes through native SDKs (Python, Node.js), a REST API, or drop-in compatibility with the OpenAI SDK by changing the base URL. Portkey holds ISO 27001, SOC 2, GDPR, and HIPAA certifications, with AES-256 encryption for data in transit and at rest. [1]

Portkey ships as both an open-source gateway (free, self-hosted) and a managed cloud service. The managed service includes a free tier of 10,000 requests per month, with paid plans for higher volumes. Enterprise customers can deploy Portkey in a private cloud configuration. [1]

## Core Concepts

**Gateway Configs** are JSON objects that define how Portkey processes requests. A config specifies the routing strategy, target providers, caching behavior, retry logic, guardrails, and request timeouts. Configs are the central orchestration mechanism for all gateway features and can be stored server-side (referenced by ID) or passed inline with each request. [2]

**Model Catalog** (formerly Virtual Keys) provides a centralized system for managing provider credentials. Instead of embedding API keys in application code, credentials are stored securely in Portkey and referenced using the `@provider-slug/model-name` syntax. The Model Catalog supports organization-level credential sharing across workspaces, fine-grained budgets, rate limits, and model allow-lists. [3]

**Targets** are the downstream LLM providers or model endpoints that receive routed requests. Each target in a config specifies a provider, credentials, optional model overrides, and weight (for load balancing). Targets can be nested to create complex routing trees with multiple fallback layers. [2]

**Strategy Modes** define how Portkey distributes requests across targets. The four modes are `single` (one provider), `loadbalance` (weighted distribution), `fallback` (sequential failover), and `conditional` (rule-based routing). [2]

**Guardrails** are real-time validators that check inputs before they reach the LLM and outputs before they reach the user. Guardrails can block requests, log violations, trigger fallbacks, or build evaluation datasets. Over 20 deterministic checks are available alongside LLM-based and third-party guardrail integrations. [4]

**Observability** is an OpenTelemetry-compliant monitoring suite that automatically captures all requests, responses, costs, latencies, and token usage. The suite includes logs, distributed tracing, analytics dashboards with 21+ metrics, custom metadata tagging, and feedback integration. [5]

## Architecture

Portkey operates as an edge-deployed proxy between client applications and LLM providers:

```
Application (SDK / REST)
    |
    v
Portkey Gateway (Edge Workers)
    |--- Guardrails (input validation)
    |--- Routing (fallback / loadbalance / conditional)
    |--- Caching (simple / semantic)
    |--- Retry Logic
    |--- Observability (logs, traces, metrics)
    |
    v
LLM Providers (OpenAI, Anthropic, Azure, Bedrock, 250+ others)
```

**Edge Infrastructure**: The gateway runs on globally distributed edge workers, minimizing latency by processing requests close to their origin. The edge layer handles routing decisions, cache lookups, guardrail evaluation, and retry logic before forwarding to the target provider. [1]

**Config-Driven Orchestration**: All gateway behavior is defined through JSON config objects. Configs can be stored server-side and referenced by ID (`pc-xxx`) or passed inline. Server-side configs enable runtime changes without code deployments. [2]

**OpenAI-Compatible Interface**: Portkey exposes an API surface compatible with the OpenAI Chat Completions format (`/v1/chat/completions`). Applications using the OpenAI SDK can switch to Portkey by changing the base URL and adding Portkey headers, with no changes to the request body. [1]

**Credential Isolation**: Provider API keys are stored in Portkey's vault (Model Catalog) and never exposed in application code. Requests reference credentials through the `@provider-slug` syntax, and Portkey injects the actual key at the edge layer before forwarding to the provider. [3]

## Key Features and Functionality

### Fallback Routing

Automatically switch to backup providers when the primary fails:

```json
{
    "strategy": {
        "mode": "fallback"
    },
    "targets": [
        {
            "virtual_key": "openai-key",
            "override_params": {"model": "gpt-4o"}
        },
        {
            "virtual_key": "anthropic-key",
            "override_params": {"model": "claude-sonnet-4-6"}
        }
    ]
}
```

Fallbacks can be triggered by specific HTTP status codes using `on_status_codes` at the target level. [2]

### Load Balancing

Distribute requests across providers or API keys using weighted targets:

```json
{
    "strategy": {
        "mode": "loadbalance"
    },
    "targets": [
        {
            "virtual_key": "openai-key-1",
            "weight": 0.7,
            "override_params": {"model": "gpt-4o"}
        },
        {
            "virtual_key": "anthropic-key",
            "weight": 0.3,
            "override_params": {"model": "claude-sonnet-4-6"}
        }
    ]
}
```

Weights control traffic distribution probability and are normalized automatically. [2]

### Conditional Routing

Route requests based on custom criteria using query conditions:

```json
{
    "strategy": {
        "mode": "conditional",
        "conditions": [
            {
                "query": {"metadata.tier": "premium"},
                "then": "target-gpt4"
            }
        ],
        "default": "target-gpt4o-mini"
    },
    "targets": [...]
}
```

### Caching

Reduce latency and costs with simple (exact match) or semantic (similarity-based) caching:

```json
{
    "cache": {
        "mode": "semantic",
        "max_age": 3600
    }
}
```

- **Simple cache**: Exact match on request body; fastest lookup
- **Semantic cache**: Similarity-based matching; returns cached responses for semantically equivalent prompts [2]

### Automatic Retries

Retry failed requests with configurable attempts and status code filters:

```json
{
    "retry": {
        "attempts": 3,
        "on_status_codes": [429, 500, 502, 503, 504],
        "use_retry_after_headers": true
    }
}
```

The `use_retry_after_headers` option respects provider-sent `Retry-After` headers for rate-limited requests. [2]

### Circuit Breaker

Prevent cascading failures by temporarily disabling unhealthy targets:

```json
{
    "cb_config": {
        "failure_threshold": 5,
        "cooldown_interval": 60000,
        "failure_status_codes": [500, 502, 503]
    }
}
```

When a target exceeds the `failure_threshold`, the circuit opens and requests are routed to other targets for the duration of the `cooldown_interval` (minimum 30 seconds). [2]

### Guardrails

Validate inputs and outputs with deterministic, LLM-based, or third-party checks:

```json
{
    "input_guardrails": ["guardrail-id-xxx"],
    "output_guardrails": ["guardrail-id-yyy"]
}
```

Guardrail actions include synchronous blocking (status 446 on failure), asynchronous logging (non-blocking), sequential or parallel execution, and feedback collection for evaluation datasets. Built-in checks cover regex matching, JSON schema validation, code detection (SQL, Python, TypeScript), prompt injection scanning, and gibberish detection. Third-party integrations with Aporia, SydeLabs, and Pillar Security are available. [4]

### Observability

All requests are automatically logged with cost, latency, token usage, and provider metadata. Features include:

- **Logs**: Full request and response capture for all multimodal interactions
- **Traces**: Distributed tracing across the lifecycle of each request
- **Analytics**: 21+ metrics on dashboards for trend analysis
- **Custom Metadata**: Tag requests with arbitrary key-value pairs for grouping and filtering
- **Feedback**: Attach feedback values and weights to close observability loops
- **Budget Limits**: Configure cost limits per provider API key [5]

### Model Context Protocol (MCP)

Connect external tools and data sources to LLM requests through MCP support, enabling agents to access databases, file systems, and APIs through a standardized protocol. [6]

## Use Cases

**Multi-Provider Resilience**: Route production traffic through Portkey with fallback configs to ensure continuity when a single provider experiences downtime. An application can fall back from OpenAI to Anthropic to Azure OpenAI without any code changes.

**Cost Optimization**: Use load balancing to distribute traffic across cheaper model tiers for routine queries while routing complex queries to frontier models via conditional routing. Semantic caching further reduces costs by serving cached responses for repeated or similar prompts.

**Compliance and Security**: Store all provider credentials in the Model Catalog, enforce guardrails on inputs and outputs to prevent prompt injection and data leakage, and enable audit logging through the observability suite. The HIPAA, SOC 2, and GDPR certifications support regulated industry deployments.

**A/B Testing and Canary Deployments**: Use weighted load balancing to gradually shift traffic from an existing model to a new model, monitoring performance and cost metrics through the analytics dashboard before full rollout.

**Agent Observability**: Trace multi-step agent workflows across multiple LLM calls, tool invocations, and retrieval steps. Custom metadata tags enable grouping traces by user session, agent type, or business workflow.

**Rate Limit Management**: Distribute requests across multiple API keys for the same provider using load balancing, effectively multiplying rate limits without application-level key rotation logic.

## API Reference Summary

### Endpoints

Portkey mirrors the OpenAI-compatible API surface:

- `POST /v1/chat/completions` -- Chat completions (text generation)
- `POST /v1/completions` -- Legacy completions
- `POST /v1/embeddings` -- Vector embeddings
- `POST /v1/images/generations` -- Image generation
- `POST /v1/audio/speech` -- Text-to-speech
- `POST /v1/audio/transcriptions` -- Speech-to-text

### Request Headers

- `x-portkey-api-key` -- Portkey API key (required)
- `x-portkey-config` -- Gateway config ID or inline JSON
- `x-portkey-virtual-key` -- Legacy virtual key reference
- `x-portkey-metadata` -- Custom metadata as JSON string
- `x-portkey-trace-id` -- Custom trace identifier
- `x-portkey-cache-namespace` -- Cache namespace for isolation
- `Authorization` -- Provider API key (when not using Model Catalog)

### Response Additions

Portkey responses include standard OpenAI-format fields plus:

- `hook_results` -- Guardrail check results (when synchronous guardrails are configured)
- Status 246: Guardrails failed but request continues
- Status 446: Guardrails failed and request is denied

## Configuration and Customization

### Complete Config Structure

```json
{
    "strategy": {
        "mode": "fallback | loadbalance | conditional | single",
        "conditions": [],
        "default": "target-id",
        "on_status_codes": [429, 500]
    },
    "targets": [
        {
            "provider": "openai",
            "api_key": "sk-...",
            "virtual_key": "key-id",
            "custom_host": "http://private-llm/v1",
            "weight": 0.7,
            "override_params": {"model": "gpt-4o", "temperature": 0.5},
            "forward_headers": ["Authorization"],
            "on_status_codes": [500, 502]
        }
    ],
    "cache": {
        "mode": "simple | semantic",
        "max_age": 3600
    },
    "retry": {
        "attempts": 3,
        "on_status_codes": [429, 500, 502, 503, 504],
        "use_retry_after_headers": true
    },
    "cb_config": {
        "failure_threshold": 5,
        "cooldown_interval": 60000,
        "failure_status_codes": [500, 502, 503]
    },
    "request_timeout": 30000,
    "input_guardrails": ["guardrail-id"],
    "output_guardrails": ["guardrail-id"],
    "strict_open_ai_compliance": true,
    "forward_headers": ["X-Custom-Header"]
}
```

### Cloud Provider Parameters

Configs support direct cloud provider authentication:

- **Azure OpenAI**: `azure_region`, `azure_deployment_name`, `azure_api_version`, `azure_endpoint_name`
- **AWS Bedrock**: `aws_access_key_id`, `aws_secret_access_key`, `aws_region`, `aws_session_token`
- **Google Vertex AI**: `vertex_project_id`, `vertex_region`, `vertex_service_account_json`

### Config Application Methods

Configs can be applied through multiple channels:

- Portkey SDK `config` parameter (inline object or stored config ID)
- OpenAI SDK via `x-portkey-config` header
- REST API via `x-portkey-config` header
- Default config attached to a Portkey API key in the dashboard

## Integration Patterns

### With OpenAI SDK (Python)

```python
from openai import OpenAI
from portkey_ai import createHeaders

client = OpenAI(
    api_key="dummy",
    base_url="https://api.portkey.ai/v1",
    default_headers=createHeaders(
        api_key="YOUR_PORTKEY_API_KEY",
        virtual_key="YOUR_OPENAI_VIRTUAL_KEY"
    )
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### With LangChain

```python
from langchain_openai import ChatOpenAI
from portkey_ai import createHeaders

llm = ChatOpenAI(
    api_key="dummy",
    base_url="https://api.portkey.ai/v1",
    default_headers=createHeaders(
        api_key="YOUR_PORTKEY_API_KEY",
        virtual_key="YOUR_OPENAI_VIRTUAL_KEY"
    ),
    model="gpt-4o"
)

response = llm.invoke("What is the meaning of life?")
```

### With Observability Tracing (Logfire)

```python
import logfire
import os
from portkey_ai import createHeaders
from openai import OpenAI

os.environ["OTEL_EXPORTER_OTLP_ENDPOINT"] = "https://api.portkey.ai/v1/logs/otel"
os.environ["OTEL_EXPORTER_OTLP_HEADERS"] = "x-portkey-api-key=YOUR_PORTKEY_API_KEY"

logfire.configure(service_name="my-llm-app", send_to_logfire=False)

client = OpenAI(
    api_key="YOUR_OPENAI_API_KEY",
    base_url="https://api.portkey.ai/v1",
    default_headers=createHeaders(
        api_key="YOUR_PORTKEY_API_KEY",
    )
)

logfire.instrument_openai(client)
```

### With Private or Self-Hosted Models

```json
{
    "strategy": {
        "mode": "fallback"
    },
    "targets": [
        {
            "provider": "openai",
            "custom_host": "http://my-private-llm:8080/v1",
            "forward_headers": ["Authorization"]
        },
        {
            "virtual_key": "openai-fallback-key"
        }
    ]
}
```

## Examples

### Production-Ready Config with Fallback, Caching, and Retries

```python
from portkey_ai import Portkey

config = {
    "strategy": {
        "mode": "fallback"
    },
    "targets": [
        {
            "virtual_key": "openai-prod",
            "override_params": {"model": "gpt-4o"},
            "weight": 1.0
        },
        {
            "virtual_key": "anthropic-prod",
            "override_params": {"model": "claude-sonnet-4-6"}
        }
    ],
    "cache": {
        "mode": "semantic",
        "max_age": 3600
    },
    "retry": {
        "attempts": 3,
        "on_status_codes": [429, 500, 502, 503, 504]
    },
    "request_timeout": 30000
}

portkey = Portkey(
    api_key="PORTKEY_API_KEY",
    config=config
)

response = portkey.chat.completions.create(
    messages=[{"role": "user", "content": "Summarize this document."}]
)
```

### Weighted Load Balancing Across Providers

```python
from portkey_ai import Portkey

config = {
    "strategy": {
        "mode": "loadbalance"
    },
    "targets": [
        {
            "virtual_key": "openai-key-1",
            "weight": 0.5,
            "override_params": {"model": "gpt-4o"}
        },
        {
            "virtual_key": "openai-key-2",
            "weight": 0.3,
            "override_params": {"model": "gpt-4o"}
        },
        {
            "virtual_key": "anthropic-key",
            "weight": 0.2,
            "override_params": {"model": "claude-sonnet-4-6"}
        }
    ]
}

portkey = Portkey(api_key="PORTKEY_API_KEY", config=config)

response = portkey.chat.completions.create(
    messages=[{"role": "user", "content": "Hello!"}]
)
```

### Embedding Request with Guardrails

```python
from portkey_ai import Portkey

portkey = Portkey(
    api_key="PORTKEY_API_KEY",
    config="pc-xxx"  # Config with embedding guardrails
)

response = portkey.embeddings.create(
    input="Your text string goes here",
    model="text-embedding-3-small"
)
```

### Custom Metadata for Observability

```python
from portkey_ai import Portkey

portkey = Portkey(
    api_key="PORTKEY_API_KEY",
)

response = portkey.with_options(
    metadata={"user_id": "user-123", "session": "abc", "environment": "production"}
).chat.completions.create(
    model="@openai-prod/gpt-4o",
    messages=[{"role": "user", "content": "Help me debug this error."}]
)
```

## Limitations and Considerations

**Added Latency**: The edge proxy adds 20-40ms of latency to every request compared to direct provider API calls. For latency-critical applications where every millisecond matters, this overhead should be evaluated against the benefits of routing and observability.

**Vendor Lock-In on Managed Features**: While the open-source gateway handles routing, caching, and retries, advanced features like the analytics dashboard, guardrails management UI, Model Catalog, and budget controls require the managed Portkey service.

**Semantic Cache Accuracy**: Semantic caching relies on similarity matching, which may return cached responses for prompts that are similar but not semantically equivalent. Applications requiring deterministic responses should use simple (exact-match) caching or disable caching entirely.

**Guardrail Latency**: Synchronous guardrails add processing time to each request. Applications with tight latency requirements should consider running guardrails asynchronously (logging only) or limiting the number of active checks per request.

**Provider Feature Parity**: Not all provider-specific features are exposed through Portkey's unified interface. Advanced or recently released provider capabilities may require direct API access until Portkey adds support.

**Virtual Key Deprecation**: Virtual Keys have been migrated to the Model Catalog system. Existing implementations using Virtual Keys continue to work but should migrate to the `@provider-slug/model-name` syntax for new projects.

**Free Tier Limits**: The managed service free tier is limited to 10,000 requests per month, which is sufficient for development but requires a paid plan for production workloads.

## Changelog Highlights

- **Model Catalog**: Replaced Virtual Keys with organization-level credential management, fine-grained budgets, rate limits, and model allow-lists
- **Guardrails on the Gateway**: Real-time input/output validation with 20+ deterministic checks, LLM-based detection, and third-party integrations (Aporia, SydeLabs, Pillar Security)
- **Conditional Routing**: Query-based routing rules for directing traffic based on custom criteria
- **Circuit Breaker**: Per-strategy failure handling with configurable thresholds and cooldown intervals
- **MCP Support**: Model Context Protocol integration for connecting external tools and data sources
- **gRPC Transport (Beta)**: Reduced-latency transport option alongside REST
- **Semantic Caching**: Similarity-based cache matching for semantically equivalent prompts
- **250+ Provider Support**: Expanded from initial provider set to over 250 supported LLM providers and models

## Citations

- [1] What is Portkey - <https://portkey.ai/docs/introduction/what-is-portkey>
- [2] Gateway Configs - <https://portkey.ai/docs/product/ai-gateway/configs>
- [3] Virtual Keys / Model Catalog - <https://portkey.ai/docs/product/ai-gateway/virtual-keys>
- [4] Guardrails - <https://portkey.ai/docs/product/guardrails>
- [5] Observability - <https://portkey.ai/docs/product/observability>
- [6] AI Gateway Overview - <https://portkey.ai/docs/product/ai-gateway>
- [7] Config Object Schema - <https://portkey.ai/docs/api-reference/inference-api/config-object>
- [8] Supported LLM Providers - <https://portkey.ai/docs/integrations/llms>
- [9] GitHub Repository - <https://github.com/Portkey-AI/gateway>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- portkey
- portkey gateway
- ai gateway
- llm gateway
- edge proxy
- gateway configs
- model catalog
- virtual keys
- targets
- strategy modes
- fallback routing
- loadbalance
- conditional routing
- semantic cache
- simple cache
- circuit breaker
- input guardrails
- output guardrails
- 250+ providers
- openai-compatible
- x-portkey-config
- x-portkey-virtual-key
- request_timeout
- cb_config
- otel observability
- 21+ metrics
- mcp on gateway
- aporia
- sydelabs
- pillar security
- iso 27001
- soc 2
- gdpr
- hipaa

### Verb-Noun Tasks

- Configure fallback routing across multiple LLM providers
- Weighted-load-balance traffic across API keys
- Apply conditional routing based on request metadata
- Cache LLM responses semantically or by exact match
- Trip a circuit breaker on a failing provider
- Enforce input and output guardrails before/after the model
- Store provider credentials in the Model Catalog instead of env vars
- Issue scoped virtual keys with budgets and allow-lists
- Stream OpenTelemetry traces to a dashboard
- Tag requests with custom metadata for trace grouping
- Add Aporia, SydeLabs, or Pillar Security as third-party guardrails
- Route requests through globally distributed edge workers
- Inject `x-portkey-config` headers into existing OpenAI SDK clients
- Forward requests to a custom self-hosted endpoint as a target

### User Intent Phrases

- How do I add automatic fallback to Anthropic when OpenAI fails?
- I want a managed gateway that runs on edge workers near my users.
- How do I cache prompts that are semantically similar?
- How do I block prompt injection at the gateway layer?
- Can I route different users to different models based on metadata?
- How do I issue rate-limited API keys to my customers?
- I need HIPAA and SOC 2 certifications for my LLM traffic.
- How do I get OpenTelemetry-compliant tracing across all my LLM calls?
- How do I A/B test two models with weighted traffic splits?
- How do I forward through Portkey to a private vLLM endpoint?
- Can I temporarily disable a provider that keeps failing?
- How do I tag every LLM request with a user_id for observability?

### Problem Statements

- Provider outages cascade into our application with no buffer.
- We have no central place to enforce safety checks across all LLM calls.
- We cannot answer "how much did each customer spend on LLMs?" today.
- Our team keeps duplicating retry, fallback, and cache logic per service.
- Compliance auditors want full request logs we currently do not have.
- Rate limits from a single API key are throttling production.
- We need to roll out a new model gradually, not flip a switch.

### When to Pick This

- Pick this when you want a managed, enterprise-grade gateway with edge deployment (vs LiteLLM's self-hosted SDK-first model).
- Pick this when you need a circuit breaker and conditional routing rules out of the box.
- Pick this when guardrails, third-party scanners, and Model Catalog credential vaulting matter more than open-source extensibility.
- Pick this when you require ISO 27001, SOC 2, GDPR, or HIPAA attestations.
- Pick this when OpenTelemetry-native observability with 21+ analytics metrics is required.
- Pick this when you need text and embeddings routing but not music/video generation (vs ccapi).
- Pick this when you need a no-code dashboard for non-engineering stakeholders.

### Related Terms and Aliases

- ai control plane
- llm api gateway
- prompt firewall
- ai proxy
- semantic caching layer
- llm routing service
- ai observability platform
- prompt observability
- portkey edge gateway
- portkey-ai/gateway
