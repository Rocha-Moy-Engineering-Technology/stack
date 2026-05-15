# Helicone

> AI gateway and LLM observability platform with routing

| Field | Value |
|-------|-------|
| Group | Observability |
| Type | API/UI |
| Open Source | Yes |
| GitHub | [https://github.com/Helicone/helicone](https://github.com/Helicone/helicone) |
| Stars | 5662 |
| Documentation | [Official Docs](https://docs.helicone.ai/) |

## Overview

Helicone is an open-source AI gateway and Large Language Model (LLM) observability platform that provides a unified, OpenAI-compatible API for accessing over 100 models across multiple providers. Backed by Y Combinator (W23) and licensed under Apache 2.0, Helicone sits between your application and LLM providers, acting as both a proxy and an observability layer. Every request that passes through the gateway is automatically logged, enabling cost tracking, latency monitoring, session tracing, and analytics without requiring changes to application logic beyond a base URL swap.

Helicone distinguishes itself from pure observability platforms by combining gateway functionality (routing, caching, rate limiting, retries, fallbacks) with a full-featured analytics dashboard. It supports a Bring Your Own Keys (BYOK) model where developers supply their own provider API keys, as well as a managed credits system with zero markup on provider pricing. The platform can be used as a hosted service at `helicone.ai` or self-hosted via Docker Compose, Kubernetes, or manual installation.

**Primary value propositions:**

- Single integration point for 100+ models from OpenAI, Anthropic, Google, Groq, Vertex AI, AWS Bedrock, and others
- Automatic logging and observability with sub-second dashboard latency
- Edge-deployed caching on Cloudflare for cost reduction and faster responses
- Production-grade features (fallbacks, retries, rate limiting) controlled entirely through HTTP headers
- SOC 2 and General Data Protection Regulation (GDPR) compliant for enterprise deployments

## Core Concepts

**AI Gateway.** The gateway is a reverse proxy that intercepts LLM API calls, logs them, and forwards them to the target provider. Requests use the standard OpenAI SDK format but point to `gateway.helicone.ai` (or `oai.helicone.ai` for OpenAI-specific traffic) instead of the provider endpoint. The target provider is specified through the `Helicone-Target-Url` header.

**Header-Driven Configuration.** Helicone's features are activated and configured entirely through HTTP headers on each request. Caching, rate limiting, retries, custom properties, session tracking, and security features are all toggled by including the appropriate `Helicone-*` headers. This design means no SDK lock-in and no configuration files to manage.

**Sessions and Traces.** Sessions group related requests (LLM calls, tool calls, vector database queries) into a single workflow view. Sessions use three headers: `Helicone-Session-Id` for the unique session identifier, `Helicone-Session-Path` for parent-child hierarchy using forward-slash syntax, and `Helicone-Session-Name` for human-readable categorization.

**Custom Properties.** Arbitrary key-value metadata attached to requests via `Helicone-Property-[Name]` headers. Properties enable segmentation by environment, feature, user tier, conversation ID, or any business dimension relevant to cost analysis and debugging.

**Proxy Mode vs. Async Mode.** Helicone supports two integration methods. Proxy mode routes requests through the gateway, providing access to all gateway features (caching, fallbacks, rate limiting). Async mode uses OpenLLMetry to log events without placing Helicone in the critical path, ensuring that Helicone issues cannot cause application outages.

**Prompt Management.** Prompts can be versioned and deployed through the gateway using `Helicone-Prompt-Id` headers, enabling iteration on prompts without code deployments.

## Architecture

Helicone's architecture comprises five core services:

```
                    +------------------+
                    |    Client App    |
                    +--------+---------+
                             |
                    OpenAI-compatible API
                             |
                    +--------v---------+
                    |  Cloudflare Edge |
                    |    (Workers)     |
                    |  - Proxy/Logging |
                    |  - Caching       |
                    |  - Rate Limiting |
                    +--------+---------+
                             |
               +-------------+-------------+
               |                           |
      +--------v---------+       +--------v---------+
      |   LLM Providers  |       |    Jawn Server   |
      | OpenAI, Anthropic|       |  (Express/Tsoa)  |
      | Google, Groq ... |       |  - Log Ingestion |
      +------------------+       |  - API Queries   |
                                 +--------+---------+
                                          |
                              +-----------+-----------+
                              |                       |
                     +--------v-------+     +---------v--------+
                     |   Supabase     |     |   ClickHouse     |
                     | - Auth         |     | - Analytics DB   |
                     | - User Data    |     | - Request Logs   |
                     +----------------+     | - Minio (Object) |
                                            +------------------+
                                                     |
                                            +--------v---------+
                                            |   Web Dashboard  |
                                            |    (Next.js)     |
                                            +------------------+
```

**Web (Next.js).** The frontend dashboard for viewing request logs, analytics, sessions, prompt management, and configuration.

**Worker (Cloudflare Workers).** The edge proxy layer that intercepts requests, applies gateway features (caching, rate limiting, retries, fallbacks), and forwards to LLM providers. Being deployed on Cloudflare's edge network provides low-latency processing globally.

**Jawn (Express/Tsoa).** The backend server responsible for log ingestion, REST API endpoints, and async event processing. Handles the write path from Workers and the read path from the dashboard and API consumers.

**Supabase.** Provides database storage for user accounts, organizations, API keys, and authentication. Handles the Identity and Access Management (IAM) layer.

**ClickHouse with Minio.** The analytics database optimized for high-volume log queries. ClickHouse handles request log storage and aggregation, while Minio provides S3-compatible object storage for request/response bodies.

The technology stack is predominantly TypeScript (91.3%), with additional MDX for documentation, Python, SQL, and shell scripting.

## Key Features and Functionality

### Request Logging and Observability

Every request through the gateway is automatically logged with full metadata: model, tokens (prompt and completion), cost, latency, status code, request body, response body, and headers. Logs appear in the dashboard within seconds and support filtering by any field.

### Edge Caching

Responses are cached on Cloudflare's edge network. Enable with a single header:

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "What is 2+2?"}],
    extra_headers={
        "Helicone-Cache-Enabled": "true",
        "Cache-Control": "max-age=3600",
    },
)
```

Cache configuration options:

- `Cache-Control`: Standard HTTP cache duration (default: 7 days)
- `Helicone-Cache-Bucket-Max-Size`: Number of different responses stored for identical requests (1-20, default: 1); useful for non-deterministic prompts
- `Helicone-Cache-Seed`: Namespace for user- or context-specific caches (e.g., `user-123`)
- `Helicone-Cache-Ignore-Keys`: Exclude specific JSON fields from cache key generation

Cache keys are computed by hashing the seed, request URL, request body, relevant headers, and bucket index. Response headers indicate cache status: `Helicone-Cache: HIT` or `Helicone-Cache: MISS`.

### Rate Limiting

Control request quotas per user, organization, or custom segment:

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello"}],
    extra_headers={
        "Helicone-RateLimit-Policy": "10;w=1000;u=cents;s=user",
    },
)
```

Policy format: `[quota];w=[time_window];u=[unit];s=[segment]`

- `quota`: Maximum allowed in the window
- `w`: Time window in seconds
- `u`: Unit of measurement (`cents`, `requests`, `tokens`)
- `s`: Segmentation key (`user`, a custom property name)

Response headers include `Helicone-RateLimit-Limit`, `Helicone-RateLimit-Remaining`, and `Helicone-RateLimit-Policy`.

### Automatic Retries

Enable exponential backoff retries for failed requests:

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello"}],
    extra_headers={
        "Helicone-Retry-Enabled": "true",
        "helicone-retry-num": "3",
        "helicone-retry-factor": "2",
    },
)
```

### Provider Fallbacks

When the primary provider fails, Helicone automatically routes to a fallback model. The `Helicone-Fallback-Index` response header indicates which fallback was used.

### Session Tracking

Group related requests into workflow traces:

```typescript
import { randomUUID } from "crypto";

const sessionId = randomUUID();

// First call in session
const step1 = await client.chat.completions.create(
  {
    model: "gpt-4o-mini",
    messages: [{ role: "user", content: "Research topic X" }],
  },
  {
    headers: {
      "Helicone-Session-Id": sessionId,
      "Helicone-Session-Path": "/research",
      "Helicone-Session-Name": "Research Pipeline",
    },
  }
);

// Child call in session
const step2 = await client.chat.completions.create(
  {
    model: "gpt-4o-mini",
    messages: [{ role: "user", content: "Summarize findings" }],
  },
  {
    headers: {
      "Helicone-Session-Id": sessionId,
      "Helicone-Session-Path": "/research/summarize",
      "Helicone-Session-Name": "Research Pipeline",
    },
  }
);
```

Path hierarchy examples:

- Workflow: `/task/research/web_search`
- Conversation: `/session/question_1/answer_1`
- Pipeline: `/process/extract/transform/load`

### User Tracking

Track per-user costs, usage patterns, and behavior:

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello"}],
    extra_headers={
        "Helicone-User-Id": "user-12345",
        "Helicone-Property-UserTier": "premium",
        "Helicone-Property-UserType": "business",
    },
)
```

The platform automatically tracks daily/weekly/monthly active users, session analytics, usage patterns, and user lifecycle stages.

### Content Moderation and Security

- `Helicone-Moderations-Enabled`: Activates OpenAI moderation on requests
- `Helicone-LLM-Security-Enabled`: Protects against prompt injection attacks

### Prompt Management

Version and deploy prompts using `Helicone-Prompt-Id` headers. Prompts can be iterated on through the dashboard and deployed to production without code changes.

### Token Overflow Handling

The `Helicone-Token-Limit-Exception-Handler` header manages context window overflow with strategies: `truncate`, `middle-out`, or `fallback`.

## Use Cases

**Multi-Provider LLM Applications.** Teams using models from multiple providers (OpenAI for chat, Anthropic for analysis, Google for embeddings) can consolidate all traffic through a single gateway, gaining unified cost tracking and the ability to switch providers by changing a single model parameter.

**Cost Optimization for Development Teams.** Caching identical requests during development and testing avoids repeated charges. Custom properties segmented by environment (`staging`, `production`) enable precise cost attribution.

**AI Agent Observability.** Sessions trace multi-step agent workflows end-to-end, revealing where agents spend time, which tool calls are most expensive, and where failures occur in complex chains.

**Per-User Cost Management in SaaS.** User ID tracking combined with rate limiting enables precise unit economics: cost per user, cost per feature, and enforcement of usage quotas by subscription tier.

**High-Availability Production Systems.** Automatic fallbacks ensure continued service when a provider experiences an outage. Retries with exponential backoff handle transient failures without application-level retry logic.

**Compliance and Auditing.** Full request/response logging with data retention policies (up to forever on Enterprise) provides an audit trail for regulated industries. GDPR-compliant EU endpoints are available at `eu.api.helicone.ai`.

## API Reference Summary

### Gateway Endpoint

All LLM requests are routed through the gateway:

```
POST https://gateway.helicone.ai/{provider-path}
POST https://ai-gateway.helicone.ai/{openai-path}
POST https://oai.helicone.ai/{openai-path}
```

The gateway accepts standard provider request formats. Set `Helicone-Target-Url` to specify the downstream provider when using `gateway.helicone.ai`.

### Query API

Retrieve logged requests programmatically:

```
POST https://api.helicone.ai/v1/request/query
```

EU endpoint: `https://eu.api.helicone.ai/v1/request/query`

**Authentication:** Bearer token via `Authorization` header.

**Request body:**

```json
{
  "filter": {
    "operator": "and",
    "left": {
      "request": {
        "model": { "contains": "gpt-4" }
      }
    },
    "right": {
      "request_response_rmt": {
        "cost": { "gt": 0.01 }
      }
    }
  },
  "limit": 100,
  "offset": 0,
  "sort": { "created_at": "desc" }
}
```

**Supported filter operators:**

- Text: `equals`, `not-equals`, `contains`, `not-contains`, `like`, `ilike`
- Numeric: `equals`, `not-equals`, `gt`, `gte`, `lt`, `lte`
- Timestamp: `equals`, `gt`, `gte`, `lt`, `lte`
- Boolean: `equals`

**Filterable fields:** `request.user_id`, `request.model`, `request.prompt`, `request.created_at`, `response.status`, `response.model`, `feedback.rating`, and `request_response_rmt` fields (latency, cost, provider, tokens, cache status). Custom property filters must be wrapped in a `request_response_rmt` object.

**Response fields per request:** `request_id`, `request_created_at`, `request_body`, `request_user_id`, `request_model`, `response_status`, `response_body`, `total_tokens`, `prompt_tokens`, `completion_tokens`, `cost`, `delay_ms`, `model`, `properties`.

### User Query API

```
POST https://api.helicone.ai/v1/user/query
```

Returns aggregated user metrics and analytics.

### Header Directory (Complete Reference)

| Header | Purpose |
|--------|---------|
| `Helicone-Auth` | Authentication (required): `Bearer <API_KEY>` |
| `Helicone-Target-URL` | Downstream provider URL |
| `Helicone-Request-Id` | Custom request UUID |
| `Helicone-User-Id` | User tracking identifier |
| `Helicone-Session-Id` | Session grouping identifier |
| `Helicone-Session-Path` | Session hierarchy path |
| `Helicone-Session-Name` | Session display name |
| `Helicone-Model-Override` | Override model for cost calculation |
| `Helicone-Prompt-Id` | Prompt version tracking |
| `Helicone-Property-[Name]` | Custom metadata properties |
| `Helicone-RateLimit-Policy` | Rate limiting configuration |
| `Helicone-Cache-Enabled` | Toggle edge caching |
| `Helicone-Cache-Seed` | Cache namespace |
| `Helicone-Cache-Bucket-Max-Size` | Max cached variants (1-20) |
| `Helicone-Cache-Ignore-Keys` | Fields to exclude from cache key |
| `Helicone-Retry-Enabled` | Toggle automatic retries |
| `helicone-retry-num` | Maximum retry attempts |
| `helicone-retry-factor` | Exponential backoff factor |
| `Helicone-Omit-Response` | Exclude response body from logs |
| `Helicone-Omit-Request` | Exclude request body from logs |
| `Helicone-Token-Limit-Exception-Handler` | Overflow strategy: `truncate`, `middle-out`, `fallback` |
| `Helicone-Moderations-Enabled` | Enable OpenAI moderation |
| `Helicone-LLM-Security-Enabled` | Enable prompt injection protection |
| `Helicone-Stream-Force-Format` | Fix stream formatting for incompatible libraries |
| `Helicone-Posthog-Key` | PostHog integration API key |
| `Helicone-Posthog-Host` | PostHog integration host |

## Configuration and Customization

### Gateway Mode Configuration

When using the generic gateway endpoint (`gateway.helicone.ai`), specify the target provider:

```python
client = OpenAI(
    base_url="https://gateway.helicone.ai/v1",
    api_key="sk-provider-key",
    default_headers={
        "Helicone-Auth": f"Bearer {os.getenv('HELICONE_API_KEY')}",
        "Helicone-Target-Url": "https://api.openai.com",
    },
)
```

For non-OpenAI providers (e.g., Groq):

```python
client = OpenAI(
    base_url="https://gateway.helicone.ai/openai/v1",
    api_key="sk-groq-key",
    default_headers={
        "Helicone-Auth": f"Bearer {os.getenv('HELICONE_API_KEY')}",
        "Helicone-Target-Url": "https://api.groq.com",
    },
)
```

### Custom Properties for Environment Segmentation

```python
client = OpenAI(
    base_url="https://ai-gateway.helicone.ai",
    api_key=os.getenv("HELICONE_API_KEY"),
    default_headers={
        "Helicone-Property-Environment": "production",
        "Helicone-Property-App": "chatbot-v2",
        "Helicone-Property-Team": "ml-engineering",
    },
)
```

### Async Logger Configuration

```python
from helicone_async import HeliconeAsyncLogger

logger = HeliconeAsyncLogger(api_key="sk-helicone-...")
logger.init()

# Set session-level properties
logger.set_properties({
    "Helicone-Session-Id": "session-abc",
    "Helicone-Session-Path": "/pipeline/step1",
    "Helicone-Session-Name": "Data Processing",
    "Helicone-Property-Pipeline": "etl-v3",
})

# Disable/enable logging dynamically
logger.disable_logging()  # Pause logging
logger.enable_logging()   # Resume logging
```

### Data Privacy Controls

Exclude sensitive content from logs:

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Process this PII data..."}],
    extra_headers={
        "Helicone-Omit-Request": "true",
        "Helicone-Omit-Response": "true",
    },
)
```

### EU Data Residency

For GDPR compliance, use EU-specific endpoints:

- API queries: `https://eu.api.helicone.ai`
- Gateway: EU-specific gateway endpoints available

## Integration Patterns

### OpenAI SDK (Direct Gateway)

The simplest integration swaps the base URL:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://ai-gateway.helicone.ai",
    api_key=os.getenv("HELICONE_API_KEY"),
)
```

### LangChain Integration

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4o-mini",
    base_url="https://gateway.helicone.ai/v1",
    api_key="sk-openai-key",
    default_headers={
        "Helicone-Auth": f"Bearer {os.getenv('HELICONE_API_KEY')}",
        "Helicone-Target-Url": "https://api.openai.com",
    },
)
```

### Vercel AI SDK

```typescript
import { createOpenAI } from "@ai-sdk/openai";

const openai = createOpenAI({
  baseURL: "https://gateway.helicone.ai/v1",
  headers: {
    "Helicone-Auth": `Bearer ${process.env.HELICONE_API_KEY}`,
    "Helicone-Target-Url": "https://api.openai.com",
  },
});
```

### LlamaIndex Integration

```python
from llama_index.llms.openai import OpenAI

llm = OpenAI(
    model="gpt-4o-mini",
    api_base="https://gateway.helicone.ai/v1",
    additional_kwargs={
        "headers": {
            "Helicone-Auth": f"Bearer {os.getenv('HELICONE_API_KEY')}",
            "Helicone-Target-Url": "https://api.openai.com",
        }
    },
)
```

### PostHog Analytics Integration

Forward LLM analytics to PostHog for product analytics correlation:

```python
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello"}],
    extra_headers={
        "Helicone-Posthog-Key": os.getenv("POSTHOG_API_KEY"),
        "Helicone-Posthog-Host": "https://app.posthog.com",
    },
)
```

### Supported Providers

**Inference providers:** OpenAI, Azure OpenAI, Anthropic, AWS Bedrock, Google Gemini, Google Vertex AI, Groq, TogetherAI, Anyscale, DeepInfra, Fireworks, Hyperbolic.

**Framework integrations:** LangChain, LlamaIndex, LangGraph, Vercel AI SDK, CrewAI, Semantic Kernel, Open WebUI.

Unapproved domains can also be proxied through the gateway with a rate limit of 10,000 requests per day.

## Examples

### Example 1: Cost-Optimized Development with Caching

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://ai-gateway.helicone.ai",
    api_key=os.getenv("HELICONE_API_KEY"),
    default_headers={
        "Helicone-Cache-Enabled": "true",
        "Cache-Control": "max-age=86400",
        "Helicone-Property-Environment": "development",
    },
)

# First call: cache MISS, billed by provider
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Explain the CAP theorem"}],
)

# Second identical call: cache HIT, zero cost, instant response
response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Explain the CAP theorem"}],
)
```

### Example 2: Multi-Step Agent Session with User Tracking

```typescript
import OpenAI from "openai";
import { randomUUID } from "crypto";

const client = new OpenAI({
  baseURL: "https://ai-gateway.helicone.ai",
  apiKey: process.env.HELICONE_API_KEY,
});

const sessionId = randomUUID();
const userId = "user-456";

// Step 1: Plan
const plan = await client.chat.completions.create(
  {
    model: "gpt-4o-mini",
    messages: [{ role: "user", content: "Plan a trip to Tokyo" }],
  },
  {
    headers: {
      "Helicone-Session-Id": sessionId,
      "Helicone-Session-Path": "/trip-planner/plan",
      "Helicone-Session-Name": "Trip Planning Agent",
      "Helicone-User-Id": userId,
      "Helicone-Property-Feature": "trip-planner",
    },
  }
);

// Step 2: Research hotels (child of plan)
const hotels = await client.chat.completions.create(
  {
    model: "gpt-4o-mini",
    messages: [
      { role: "user", content: "Find hotels in Shinjuku under $200/night" },
    ],
  },
  {
    headers: {
      "Helicone-Session-Id": sessionId,
      "Helicone-Session-Path": "/trip-planner/plan/hotels",
      "Helicone-Session-Name": "Trip Planning Agent",
      "Helicone-User-Id": userId,
      "Helicone-Property-Feature": "trip-planner",
    },
  }
);

// Step 3: Summarize itinerary
const summary = await client.chat.completions.create(
  {
    model: "gpt-4o-mini",
    messages: [{ role: "user", content: "Summarize the complete itinerary" }],
  },
  {
    headers: {
      "Helicone-Session-Id": sessionId,
      "Helicone-Session-Path": "/trip-planner/summarize",
      "Helicone-Session-Name": "Trip Planning Agent",
      "Helicone-User-Id": userId,
      "Helicone-Property-Feature": "trip-planner",
    },
  }
);
```

### Example 3: Rate-Limited SaaS with Per-User Quotas

```python
import os
from openai import OpenAI

def create_client_for_user(user_id: str, tier: str) -> OpenAI:
    # 100 cents per 3600 seconds for free tier, 1000 for premium
    quota = "100" if tier == "free" else "1000"
    return OpenAI(
        base_url="https://ai-gateway.helicone.ai",
        api_key=os.getenv("HELICONE_API_KEY"),
        default_headers={
            "Helicone-User-Id": user_id,
            "Helicone-Property-UserTier": tier,
            "Helicone-RateLimit-Policy": f"{quota};w=3600;u=cents;s=user",
        },
    )

client = create_client_for_user("user-789", "free")

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello!"}],
)
```

### Example 4: Querying Logged Data via REST API

```python
import requests
import os

response = requests.post(
    "https://api.helicone.ai/v1/request/query",
    headers={
        "Authorization": f"Bearer {os.getenv('HELICONE_API_KEY')}",
        "Content-Type": "application/json",
    },
    json={
        "filter": {
            "operator": "and",
            "left": {
                "request": {
                    "model": {"contains": "gpt-4"}
                }
            },
            "right": {
                "request_response_rmt": {
                    "cost": {"gt": 0.05}
                }
            },
        },
        "limit": 50,
        "offset": 0,
        "sort": {"created_at": "desc"},
    },
)

data = response.json()
for req in data["data"]:
    print(f"Model: {req['model']}, Cost: ${req['cost']:.4f}, "
          f"Latency: {req['delay_ms']}ms, Tokens: {req['total_tokens']}")
```

### Example 5: Async Logging with Session Properties

```python
from helicone_async import HeliconeAsyncLogger
from openai import OpenAI

logger = HeliconeAsyncLogger(api_key="sk-helicone-...")
logger.init()

client = OpenAI(api_key="sk-openai-...")

# Set properties for the next group of calls
logger.set_properties({
    "Helicone-Session-Id": "batch-job-001",
    "Helicone-Session-Path": "/batch/process",
    "Helicone-Session-Name": "Nightly Batch",
    "Helicone-Property-Pipeline": "content-generation",
})

response = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Generate product description"}],
)

# Temporarily disable logging for internal calls
logger.disable_logging()
# ... internal calls not logged ...
logger.enable_logging()
```

## Limitations and Considerations

**Latency Overhead.** Proxy mode adds a small latency overhead since requests route through Cloudflare Workers before reaching the provider. For latency-sensitive applications, the async integration avoids this by logging out of band.

**Free Tier Constraints.** The free (Hobby) tier is limited to 10,000 requests per month, 1 GB storage, 7-day data retention, and 10 logs per minute ingestion. Production workloads will likely require the Pro tier ($79/month) or higher.

**Data Retention.** Retention is tier-dependent: 7 days (Free), 1 month (Pro), 3 months (Team), and unlimited (Enterprise). Historical data beyond the retention window is permanently deleted.

**Rate Limit on Unapproved Domains.** Providers not in Helicone's approved list can still be proxied, but are subject to a 10,000 requests per day limit.

**Custom Property Query Caveat.** When querying the REST API, custom property filters must be wrapped in a `request_response_rmt` object. Omitting this wrapper silently returns empty results, which can be a source of debugging confusion.

**Self-Hosting Complexity.** While open-source, the self-hosted deployment requires running five services (Next.js, Cloudflare Workers, Express, Supabase, ClickHouse with Minio). Docker Compose simplifies local development, but production self-hosting demands significant infrastructure expertise.

**Feature Parity Between Modes.** The async (OpenLLMetry) integration does not support gateway features like caching, rate limiting, retries, and fallbacks. These features require proxy mode since they operate at the request routing layer.

**No Native SDK.** Helicone relies on the OpenAI SDK with base URL modification rather than providing a dedicated client SDK. While this reduces vendor lock-in, it means feature configuration is done entirely through HTTP headers rather than typed SDK methods.

## Changelog Highlights

- **November 2025** -- Claude Sonnet 4 and Claude Sonnet 4.5 models on the AI Gateway support 1M context window by default
- **August 2025** -- Reasoning effort control added to Playground with adjustable reasoning parameters and a "minimal" option for faster responses
- **August 2025** -- GPT-5 model family pricing support across multiple providers
- **August 2025** -- Pricing support for GPT-OSS models (Fireworks, Groq, OpenRouter) and Claude Opus 4.1
- **July 2025** -- Prompt Management V2 launched with composability, version control, and instant deployment through the AI Gateway via prompt IDs
- **July 2025** -- Automatic timezone detection and country-based request filtering powered by Cloudflare's edge network

## Citations

- Helicone Documentation: [https://docs.helicone.ai/](https://docs.helicone.ai/)
- Helicone GitHub Repository: [https://github.com/Helicone/helicone](https://github.com/Helicone/helicone)
- Helicone Changelog: [https://www.helicone.ai/changelog](https://www.helicone.ai/changelog)
- Helicone Pricing: [https://www.helicone.ai/pricing](https://www.helicone.ai/pricing)
- Helicone Header Directory: [https://docs.helicone.ai/helicone-headers/header-directory](https://docs.helicone.ai/helicone-headers/header-directory)
- Helicone Gateway Integration: [https://docs.helicone.ai/getting-started/integration-method/gateway](https://docs.helicone.ai/getting-started/integration-method/gateway)
- Helicone OpenLLMetry Integration: [https://docs.helicone.ai/getting-started/integration-method/openllmetry](https://docs.helicone.ai/getting-started/integration-method/openllmetry)
- Helicone REST API Reference: [https://docs.helicone.ai/rest/request/post-v1requestquery](https://docs.helicone.ai/rest/request/post-v1requestquery)

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- Helicone
- AI gateway
- LLM proxy
- OpenAI-compatible
- base URL swap
- header-driven configuration
- edge caching
- Cloudflare Workers
- rate limiting
- automatic retries
- provider fallbacks
- session tracking
- user tracking
- custom properties
- prompt management
- async logger
- OpenLLMetry
- BYOK
- 100+ models
- Y Combinator
- Apache 2.0
- SOC 2
- GDPR
- EU endpoint
- request query API
- ClickHouse
- Supabase

### Verb-Noun Tasks

- Swap base URL to gateway.helicone.ai to start logging
- Enable edge caching with a single header
- Configure per-user rate limits via Helicone-RateLimit-Policy header
- Track sessions with Helicone-Session-Id and Session-Path
- Tag requests with custom Helicone-Property-* metadata
- Add automatic retries with exponential backoff
- Fall back to a secondary provider on failure
- Query logged requests via the REST API
- Forward analytics to PostHog
- Hide sensitive request/response bodies via Omit headers
- Use EU endpoint for GDPR compliance
- Swap to async OpenLLMetry mode to keep Helicone off the critical path

### User Intent Phrases

- I want LLM observability without changing application code.
- How do I add caching to LLM API calls to save money?
- I need per-user rate limits for a SaaS product.
- How do I track multi-step agent workflows as sessions?
- I want fallback routing when an LLM provider is down.
- How do I get unified logging across OpenAI, Anthropic, Groq, Bedrock?
- I need an LLM proxy that supports BYOK with no markup.
- How do I add prompt injection protection at the gateway level?
- Show me how to filter LLM logs by cost or user via REST API.
- I need GDPR-compliant LLM logging in the EU.

### Problem Statements

- Adding observability requires invasive SDK instrumentation everywhere.
- LLM costs balloon when identical requests aren't cached.
- We can't enforce per-user quotas without writing our own throttling layer.
- Provider outages bring down our whole product.
- Spread of providers (OpenAI, Anthropic, Groq) makes unified cost tracking hard.
- Sensitive prompts must be excluded from logs for compliance.

### When to Pick This

- Pick this when you want zero-code observability via a single base URL swap.
- Pick this over LangSmith/Langfuse/Phoenix when gateway features (caching, rate limits, fallbacks, retries) matter as much as logging.
- Pick this over LiteLLM when you also want a hosted analytics dashboard and edge caching, not just a routing SDK.
- Pick this over Portkey when you want open-source self-hosting (Apache 2.0) with the same proxy model.
- Pick this over Weights & Biases when LLM proxy/gateway concerns dominate over experiment tracking.
- Pick this when you need EU data residency for LLM logs.

### Related Terms and Aliases

- Helicone-Auth header
- Helicone-Target-Url
- ai-gateway.helicone.ai
- oai.helicone.ai
- gateway.helicone.ai
- eu.api.helicone.ai
- Jawn server
- async logging (OpenLLMetry)
- LLM API gateway
- LLM cost dashboard
- BYOK proxy
- Hobby/Pro/Team/Enterprise tiers

