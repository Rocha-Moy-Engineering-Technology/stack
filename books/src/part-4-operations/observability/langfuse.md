# Langfuse

> Open-source LLM engineering platform with observability and prompt management

| Field | Value |
|-------|-------|
| Group | Observability |
| Type | API/SDK/UI |
| Open Source | Yes |
| GitHub | [https://github.com/langfuse/langfuse](https://github.com/langfuse/langfuse) |
| Stars | 27209 |
| Documentation | [Official Docs](https://langfuse.com/docs) |

## Overview

Langfuse is an open-source LLM engineering platform that provides comprehensive observability, prompt management, and evaluation capabilities for Large Language Model (LLM) applications. Designed for teams that need to collaboratively debug, analyze, and iterate on their LLM-powered systems, Langfuse captures detailed traces of every request flowing through an application, including LLM calls, retrieval operations, embedding generation, tool executions, and arbitrary API interactions. [1]

The platform is built on OpenTelemetry standards, reducing vendor lock-in while providing LLM-specific instrumentation that general-purpose observability tools lack. Langfuse natively understands token usage, model parameters, prompt/completion pairs, and evaluation scores. It is available as a managed cloud service (with EU and US regions) or as a fully self-hostable deployment under the MIT license, making it suitable for organizations with strict data residency and privacy requirements. [1][2]

Langfuse spans three primary capability areas: **Observability** for tracing and debugging LLM application execution flows, **Prompt Management** for versioning, testing, and deploying prompts without code changes, and **Evaluation** for assessing output quality through automated, human, and model-based scoring methods. [1]

## Core Concepts

### Traces

A trace represents the complete lifecycle of a single request as it flows through an LLM application. Each trace captures the full execution path, from initial input to final output, with timing, cost, and metadata at every step. Traces are the fundamental unit of observability in Langfuse and serve as the container for all nested observations. [3]

### Observations

Observations are the building blocks within a trace. They represent individual operations such as LLM calls, retrieval steps, tool executions, or custom logic. Observations can be nested to reflect the hierarchical structure of an application. There are two primary observation types:

- **Spans**: Generic operations representing any unit of work (data processing, API calls, retrieval steps)
- **Generations**: LLM-specific operations that capture model name, token usage, prompt/completion pairs, and cost information [3]

### Sessions

Sessions group multiple traces together to represent multi-turn conversations or related interactions from the same user. This enables tracking of conversation flows across multiple requests, making it possible to analyze user journeys and multi-step agent workflows. [3]

### Scores

Scores are evaluation results attached to traces or observations. They come in three types: **numeric** (continuous values like 0-1 quality ratings), **boolean** (pass/fail assessments), and **categorical** (classification labels). Scores can originate from automated LLM-as-a-judge evaluators, user feedback collected through the application frontend, manual human annotation, or programmatic custom metrics. [5]

### Datasets

Datasets are collections of input and expected output pairs used for systematic testing. Each dataset item contains an input (any structured object), an optional expected output for comparison, and optional metadata. Datasets can be populated from production traces where application performance was suboptimal, enabling a feedback loop from production to development. [7]

### Prompts

Prompts in Langfuse are versioned, managed artifacts that can be deployed to production via labels without code changes. They come in two types: **text prompts** (single string templates) and **chat prompts** (arrays of message objects with roles). Both support variable interpolation using `{{variable}}` syntax. [6]

## Architecture

### System Components

Langfuse consists of two primary application containers backed by a multi-database storage layer:

```
                    ┌─────────────────────┐
                    │    Langfuse Web      │
                    │  (UI + REST API)     │
                    └────────┬────────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼───┐  ┌──────▼─────┐  ┌────▼────────┐
     │ PostgreSQL  │  │ ClickHouse │  │ Redis/Valkey│
     │(operational)│  │  (OLAP)    │  │  (cache)    │
     └─────────────┘  └────────────┘  └─────────────┘
                             │
                    ┌────────▼────────────┐
                    │   Langfuse Worker   │
                    │ (async processing)  │
                    └─────────────────────┘
                             │
                    ┌────────▼────────────┐
                    │   S3/Blob Storage   │
                    │  (event persistence)│
                    └─────────────────────┘
```

- **Langfuse Web**: Serves the dashboard UI and all REST API endpoints
- **Langfuse Worker**: Handles asynchronous event processing, LLM-as-a-judge evaluations, batch exports, and background migrations
- **PostgreSQL**: Stores operational data (projects, API keys, prompt definitions, dataset configurations)
- **ClickHouse**: High-performance OLAP database optimized for storing and querying traces, observations, and scores at scale
- **Redis/Valkey**: Provides in-memory caching for API key validation, prompt caching, and job queue management
- **S3/Blob Storage**: Persists incoming trace events and stores multi-modal attachments, providing recoverability if databases are temporarily unavailable [8]

### Data Flow

Trace events are queued locally in the SDK and flushed in batches asynchronously, ensuring that application response times are not affected by observability overhead. On the server side, incoming events are persisted to S3 before being processed into ClickHouse, preventing data loss during traffic spikes. API key caching in Redis reduces database load, and prompt caching accelerates SDK responses for frequently fetched prompts. [3][8]

### OpenTelemetry Foundation

Langfuse is built on OpenTelemetry, the industry-standard observability framework. The Python SDK uses the `@observe` decorator which creates OpenTelemetry spans under the hood. The JavaScript/TypeScript SDK integrates directly with the OpenTelemetry `NodeSDK` through a custom `LangfuseSpanProcessor`. This architecture enables interoperability with existing OpenTelemetry-instrumented services and reduces vendor lock-in. [2]

## Key Features and Functionality

### Tracing and Observability

Langfuse provides comprehensive tracing that captures the full execution flow of LLM applications. Traces include execution timelines for latency debugging, cost and token usage dashboards, and agent graph visualization for complex workflows:

```python
from langfuse import observe, get_client

langfuse = get_client()

@observe()
def rag_pipeline(query: str):
    documents = retrieve_documents(query)
    response = generate_answer(query, documents)
    return response

@observe()
def retrieve_documents(query: str):
    # Automatically captured as a child span
    embeddings = embed_query(query)
    return vector_search(embeddings)

@observe(as_type="generation")
def generate_answer(query: str, context: list):
    # Captured as a generation with LLM-specific metadata
    with langfuse.start_as_current_observation(
        as_type="generation",
        name="answer-generation",
        model="gpt-4o",
        input=[{"role": "user", "content": query}]
    ) as generation:
        answer = call_openai(query, context)
        generation.update(
            output=answer,
            usage_details={"input_tokens": 150, "output_tokens": 80}
        )
    return answer
```
[3][4]

### User and Session Tracking

Associate traces with users and sessions for multi-turn conversation analysis:

```python
from langfuse import observe, propagate_attributes

@observe()
def handle_request(user_id: str, session_id: str, query: str):
    with propagate_attributes(
        user_id=user_id,
        session_id=session_id,
        metadata={"source": "api"},
        tags=["production", "v2"]
    ):
        result = process_query(query)
        return result
```
[4]

### Prompt Management

Create, version, and deploy prompts through the UI, SDK, or REST API. Prompts are deployed to production via labels, enabling non-code prompt updates:

```python
from langfuse import get_client

langfuse = get_client()

# Fetch the production version of a prompt
prompt = langfuse.get_prompt("greeting-prompt", label="production")

# Compile with variables
compiled = prompt.compile(name="Alice", topic="weather")

# Use the compiled prompt with your LLM
response = call_llm(compiled)
```

```python
# Create a new prompt version programmatically
langfuse.create_prompt(
    name="greeting-prompt",
    prompt="Hello {{name}}! Let me help you with {{topic}}.",
    config={"temperature": 0.7, "model": "gpt-4o"},
    labels=["staging"]
)
```

Creating a prompt with an existing name automatically generates a new version rather than overwriting. The `production` label is used to designate the version that should be fetched by default in production environments. Prompt performance can be tracked by linking prompts to traces, enabling analysis of quality metrics across prompt versions. [6]

### LLM Playground

The built-in LLM Playground allows interactive testing of prompts with different models, parameters, and variable values directly in the Langfuse UI. This enables rapid iteration without writing code or redeploying the application. [1]

### LLM-as-a-Judge Evaluation

Configure automated evaluators that use an LLM to assess the quality of your application's outputs. Evaluators can target individual observations (completing in seconds), full traces (completing in minutes), or experiment batches:

```python
from langfuse import get_client

langfuse = get_client()

# Fetch a dataset for evaluation
dataset = langfuse.get_dataset("qa-test-cases")

def task(*, item, **kwargs):
    question = item.input
    response = call_llm(question)
    return response

# Run experiment with automatic LLM-as-a-judge evaluation
result = dataset.run_experiment(
    name="GPT-4o QA Evaluation",
    description="Testing QA quality with LLM-as-a-judge",
    task=task
)
langfuse.flush()
```

Langfuse provides managed evaluator templates for common dimensions (hallucination detection, context relevance, toxicity, helpfulness) and supports custom evaluator prompts with configurable scoring ranges. Evaluator results include both numerical scores and reasoning explanations. [5]

### Annotation Queues

Annotation queues streamline the process of human review for large batches of traces, sessions, and observations. Teams create queues with specific scoring criteria, and reviewers work through items systematically, providing human baseline scores that complement automated evaluations. [5]

### Datasets and Experiments

Datasets enable systematic, reproducible testing of LLM applications. Items can be created manually, populated from production traces, or added via the SDK:

```python
langfuse = get_client()

# Create a dataset
langfuse.create_dataset(
    name="evaluation/geography-qa",
    description="Geography question-answer pairs",
    metadata={"domain": "geography"}
)

# Add items to the dataset
langfuse.create_dataset_item(
    dataset_name="evaluation/geography-qa",
    input={"question": "What is the capital of France?"},
    expected_output={"answer": "Paris"},
    metadata={"difficulty": "easy"}
)
```

Dataset names support folder-like organization using forward slashes (e.g., `evaluation/geography-qa`). Optional JSON Schema validation can enforce data quality on dataset items. Each modification to a dataset item generates a new version with timestamp tracking. [7]

### Cost and Token Tracking

Langfuse automatically tracks token usage and cost across all LLM calls, providing dashboards for monitoring spend by model, user, session, or time period. This is captured natively through generation observations. [3]

## Use Cases

### RAG Application Debugging

Trace the full Retrieval-Augmented Generation pipeline from query embedding through vector search, document retrieval, and final generation. Identify bottlenecks in retrieval quality, measure latency at each stage, and evaluate answer quality against expected outputs using datasets.

### Agent Workflow Monitoring

Visualize complex agent execution graphs with tool calls, reasoning steps, and decision branches. Track multi-step agent workflows across sessions, monitor tool usage patterns, and identify failure modes in autonomous agent behavior.

### Prompt Engineering and Optimization

Use the prompt management system to A/B test prompt variations. Deploy new prompt versions via labels, track performance metrics per version, and use the LLM Playground for rapid iteration before promoting changes to production.

### Production Quality Monitoring

Configure LLM-as-a-judge evaluators to continuously assess output quality on live production traces. Set up annotation queues for human review of flagged outputs. Combine automated and human scores to build a comprehensive quality picture.

### Cost Optimization

Analyze token usage and cost dashboards to identify expensive operations. Compare model performance across tiers (e.g., GPT-4o versus GPT-4o-mini) using datasets and experiments to find the optimal cost-quality tradeoff.

### Multi-Turn Conversation Analysis

Group related traces into sessions to analyze complete conversation flows. Track user satisfaction across multi-turn interactions, identify conversation abandonment patterns, and measure cumulative cost per conversation.

## API Reference Summary

### REST API Endpoints

Langfuse exposes a REST API authenticated via HTTP Basic Auth using the public and secret key pair:

```bash
# Authentication format
curl -u "pk-lf-...:sk-lf-..." "https://cloud.langfuse.com/api/public/..."
```

Key endpoints:

- `POST /api/public/ingestion` -- Ingest trace events (primary SDK endpoint)
- `GET /api/public/traces` -- List traces with filtering
- `GET /api/public/traces/{traceId}` -- Retrieve a specific trace
- `GET /api/public/observations` -- List observations
- `POST /api/public/scores` -- Create a score on a trace or observation
- `GET /api/public/scores` -- List scores with filtering
- `GET /api/public/v2/prompts` -- List all prompts
- `GET /api/public/v2/prompts/{promptName}` -- Get a specific prompt
- `POST /api/public/v2/prompts` -- Create a new prompt version
- `GET /api/public/v2/datasets` -- List datasets
- `POST /api/public/v2/datasets` -- Create a dataset
- `POST /api/public/v2/dataset-items` -- Create a dataset item
- `GET /api/public/v2/datasets/{datasetName}/runs` -- List experiment runs [6][7]

### SDK Methods (Python)

- `get_client()` -- Access the globally initialized Langfuse client
- `@observe()` -- Decorator for automatic trace/span creation
- `langfuse.get_prompt(name, label)` -- Fetch a managed prompt
- `langfuse.create_prompt(name, prompt, config, labels)` -- Create a prompt version
- `langfuse.create_dataset(name, description, metadata)` -- Create a dataset
- `langfuse.create_dataset_item(dataset_name, input, expected_output)` -- Add dataset item
- `langfuse.get_dataset(name)` -- Fetch a dataset
- `dataset.run_experiment(name, task)` -- Run an experiment against a dataset
- `langfuse.update_current_trace(input, output)` -- Update the active trace
- `langfuse.start_as_current_observation(as_type, name, model)` -- Create a nested observation
- `langfuse.flush()` -- Flush all pending events to the server
- `propagate_attributes(user_id, session_id, metadata, tags)` -- Set trace-level attributes [4]

### SDK Methods (JavaScript/TypeScript)

- `LangfuseSpanProcessor` -- OpenTelemetry span processor for Langfuse
- `startActiveObservation(name, callback)` -- Create a traced observation with automatic context
- `startObservation(name, attributes, options)` -- Create a manual observation
- `propagateAttributes(attributes, callback)` -- Propagate trace attributes to children
- `updateActiveTrace(attributes)` -- Update the current trace
- `observeOpenAI(client)` -- Wrap OpenAI client for automatic tracing [2]

## Configuration and Customization

### Environment Variables

- **`LANGFUSE_SECRET_KEY`** -- Secret key for API authentication (required)
- **`LANGFUSE_PUBLIC_KEY`** -- Public key for API authentication (required)
- **`LANGFUSE_BASE_URL`** -- API base URL; defaults to `https://cloud.langfuse.com`
- **`LANGFUSE_OBSERVE_DECORATOR_IO_CAPTURE_ENABLED`** -- Toggle input/output capture globally (Python)

### Decorator Configuration (Python)

- **`name`** -- Custom identifier for the observation (defaults to function name)
- **`as_type`** -- Observation type: `"span"` (default) or `"generation"` for LLM calls
- **`capture_input`** -- Whether to record function arguments (default `True`)
- **`capture_output`** -- Whether to record function return values (default `True`)

```python
@observe(name="llm-call", as_type="generation", capture_input=True, capture_output=True)
def my_llm_function(prompt: str):
    return call_llm(prompt)
```
[4]

### Trace Attributes

- **`user_id`** -- Associate traces with individual users for user-level analytics
- **`session_id`** -- Group traces into sessions for multi-turn conversation tracking
- **`metadata`** -- Arbitrary key-value pairs for custom filtering and organization
- **`tags`** -- String labels for categorizing and filtering traces
- **`environments`** -- Separate traces across application stages (development, staging, production) [3]

### Self-Hosted Configuration

Self-hosted deployments support configuration of authentication/SSO, encryption at rest, data masking, custom base paths, transactional email, and OpenTelemetry export for infrastructure-level observability. Enterprise editions add UI customization and instance management APIs. [8]

## Integration Patterns

### OpenAI SDK (Drop-in Replacement)

Langfuse provides drop-in wrapper functions for the OpenAI SDK that automatically trace all LLM calls without changing application code:

```python
from langfuse.openai import openai

# All OpenAI calls are automatically traced
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello!"}]
)
```

```typescript
import { OpenAI } from "openai";
import { observeOpenAI } from "@langfuse/openai";

const openai = observeOpenAI(new OpenAI());
// All calls are automatically traced
const response = await openai.chat.completions.create({
  model: "gpt-4o",
  messages: [{ role: "user", content: "Hello!" }]
});
```
[2]

### LangChain

Langfuse integrates with LangChain through a callback handler that captures chain and agent execution:

```python
from langfuse.callback import CallbackHandler

langfuse_handler = CallbackHandler()

# Pass to any LangChain chain or agent
result = chain.invoke(
    {"input": "What is the weather?"},
    config={"callbacks": [langfuse_handler]}
)
```
[1]

### LlamaIndex

Langfuse provides a callback handler for LlamaIndex to capture query engine and retrieval operations:

```python
from langfuse import get_client

langfuse = get_client()

# LlamaIndex integration via Settings
from llama_index.core import Settings
Settings.callback_manager = langfuse.get_llama_index_handler()
```
[1]

### Vercel AI SDK

Enable OpenTelemetry tracing in the Vercel AI SDK to send spans to Langfuse:

```typescript
import { generateText } from "ai";
import { openai } from "@ai-sdk/openai";

const result = await generateText({
  model: openai("gpt-4.1"),
  prompt: "Write a short story about a cat.",
  experimental_telemetry: {
    isEnabled: true,
    functionId: "my-function",
    metadata: {
      sessionId: "123",
      userId: "456",
      tags: ["production"],
    },
  },
});
```
[2]

### LLM Gateways (LiteLLM, Portkey)

Langfuse integrates with LLM gateway/proxy layers that provide unified access to multiple LLM providers. Gateways can forward trace data to Langfuse for centralized observability across all provider calls. [1]

### OpenTelemetry Native

Any application instrumented with OpenTelemetry can send traces to Langfuse through the OpenTelemetry protocol (OTLP), enabling integration from any programming language that supports OpenTelemetry. [2]

## Examples

### Full RAG Pipeline with Tracing

```python
from langfuse import observe, get_client, propagate_attributes

langfuse = get_client()

@observe()
def rag_pipeline(user_id: str, session_id: str, query: str):
    with propagate_attributes(user_id=user_id, session_id=session_id):
        documents = retrieve(query)
        answer = generate(query, documents)
        langfuse.update_current_trace(
            input={"query": query},
            output={"answer": answer}
        )
        return answer

@observe()
def retrieve(query: str):
    embedding = embed(query)
    results = vector_search(embedding, top_k=5)
    return results

@observe(as_type="generation")
def generate(query: str, context: list):
    with langfuse.start_as_current_observation(
        as_type="generation",
        name="answer-llm",
        model="gpt-4o",
        input=[
            {"role": "system", "content": "Answer based on the provided context."},
            {"role": "user", "content": f"Context: {context}\n\nQuestion: {query}"}
        ]
    ) as generation:
        answer = call_openai(query, context)
        generation.update(
            output=answer,
            usage_details={"input_tokens": 500, "output_tokens": 150}
        )
    return answer

rag_pipeline("user_42", "session_abc", "What are the benefits of RAG?")
langfuse.flush()
```

### Prompt Management Workflow

```python
from langfuse import get_client

langfuse = get_client()

# Create initial prompt version
langfuse.create_prompt(
    name="qa-system-prompt",
    type="chat",
    prompt=[
        {"role": "system", "content": "You are a helpful assistant. Answer based on the context provided."},
        {"role": "user", "content": "Context: {{context}}\n\nQuestion: {{question}}"}
    ],
    config={"temperature": 0.3, "model": "gpt-4o"},
    labels=["staging"]
)

# Fetch and use in production
prompt = langfuse.get_prompt("qa-system-prompt", label="production")
messages = prompt.compile(context="Langfuse is an LLM platform.", question="What is Langfuse?")

# Call LLM with managed prompt
response = call_llm(messages)
```

### Dataset-Driven Experiment

```python
from langfuse import get_client
from openai import OpenAI

langfuse = get_client()
openai_client = OpenAI()

# Create a dataset
langfuse.create_dataset(name="qa-golden-set")
langfuse.create_dataset_item(
    dataset_name="qa-golden-set",
    input={"question": "What is the capital of France?"},
    expected_output={"answer": "Paris"}
)
langfuse.create_dataset_item(
    dataset_name="qa-golden-set",
    input={"question": "What is the largest ocean?"},
    expected_output={"answer": "Pacific Ocean"}
)

# Define the task function
def qa_task(*, item, **kwargs):
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": item.input["question"]}]
    )
    return response.choices[0].message.content

# Run the experiment
dataset = langfuse.get_dataset("qa-golden-set")
result = dataset.run_experiment(
    name="GPT-4o Baseline",
    description="Baseline QA evaluation with GPT-4o",
    task=qa_task
)
langfuse.flush()
```

### REST API Prompt Management

```bash
# List all prompts
curl -u "pk-lf-...:sk-lf-..." \
  "https://cloud.langfuse.com/api/public/v2/prompts"

# Get a specific prompt by name
curl -u "pk-lf-...:sk-lf-..." \
  "https://cloud.langfuse.com/api/public/v2/prompts/greeting-prompt"

# Create a new prompt version
curl -X POST \
  -u "pk-lf-...:sk-lf-..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "greeting-prompt",
    "prompt": "Hello {{name}}! Let me help you with {{topic}}.",
    "config": {"temperature": 0.7},
    "labels": ["production"]
  }' \
  "https://cloud.langfuse.com/api/public/v2/prompts"
```

## Limitations and Considerations

- **Asynchronous ingestion**: Trace events are queued and flushed in batches; in short-lived processes (serverless functions, scripts), `langfuse.flush()` must be called explicitly before shutdown to avoid data loss [3]
- **ClickHouse dependency**: Self-hosted deployments require ClickHouse in addition to PostgreSQL, adding operational complexity compared to single-database platforms [8]
- **LLM-as-a-judge cost**: Automated model-based evaluations incur additional LLM API costs for the judge model; high-capability models (GPT-4o, Claude Sonnet) are recommended for scoring accuracy, increasing evaluation expenses [5]
- **Evaluation latency**: Trace-level LLM-as-a-judge evaluations can take minutes to complete due to the complexity of assembling full trace context for the judge model [5]
- **OpenTelemetry complexity**: The JavaScript/TypeScript SDK requires explicit OpenTelemetry setup (NodeSDK, span processors), which adds initialization boilerplate compared to the Python decorator approach [2]
- **Self-hosting infrastructure**: Production-grade self-hosted deployments require PostgreSQL, ClickHouse, Redis/Valkey, and S3-compatible storage, representing a significant infrastructure footprint [8]
- **Prompt type immutability**: Once a prompt is created as either text or chat type, the type cannot be changed; a new prompt with a different name must be created instead [6]

## Changelog Highlights

- **All product features open-sourced** (MIT license): LLM-as-a-judge evaluations, annotation queues, prompt experiments, and the LLM Playground are all available for self-hosted deployments [1]
- **Langfuse v3 stable release**: New architecture with ClickHouse for OLAP queries, S3-based event persistence, and the worker container for scalable async processing [8]
- **OpenTelemetry foundation**: SDK rebuilt on OpenTelemetry standards for cross-language interoperability and reduced vendor lock-in [2]
- **Python SDK v3**: Generally available with `@observe` decorator, `get_client()` global access, and `start_as_current_observation` context manager [4]
- **JavaScript/TypeScript SDK v4**: OpenTelemetry-native with `LangfuseSpanProcessor`, `startActiveObservation`, and `observeOpenAI` wrapper [2]
- **Prompt experiments**: Run experiments against datasets directly within Langfuse for systematic prompt evaluation [7]
- **Managed evaluators**: Pre-built LLM-as-a-judge templates for hallucination, context relevance, toxicity, and helpfulness [5]
- **50+ framework integrations**: Including OpenAI, LangChain, LlamaIndex, Vercel AI SDK, LiteLLM, and OpenTelemetry native support [1]
- **Dataset schema validation**: Optional JSON Schema enforcement on dataset items for data quality [7]
- **Batch exports**: Export large volumes of trace data as CSV/JSON from the UI [8]

## Citations

- [1] Langfuse Documentation - <https://langfuse.com/docs>
- [2] Get Started Guide - <https://langfuse.com/docs/get-started>
- [3] Tracing Overview - <https://langfuse.com/docs/tracing>
- [4] Python SDK Decorators - <https://langfuse.com/docs/sdk/python/decorators>
- [5] Evaluation Overview - <https://langfuse.com/docs/scores/overview>
- [6] Prompt Management - <https://langfuse.com/docs/prompts/get-started>
- [7] Datasets Overview - <https://langfuse.com/docs/datasets/overview>
- [8] Self-Hosting Guide - <https://langfuse.com/docs/deployment/self-host>

## Discovery Signals

> Keywords, phrases, and user-intent patterns that should surface this chapter in semantic search.

### Keywords

- llm observability
- open source observability
- prompt management
- tracing
- spans
- generations
- sessions
- scores
- datasets
- experiments
- llm-as-a-judge
- evaluation
- annotation queues
- cost tracking
- token usage
- OpenTelemetry
- @observe decorator
- self-hosted
- MIT license
- ClickHouse
- PostgreSQL
- Langfuse
- prompt versioning
- LLM Playground
- multi-turn conversations

### Verb-Noun Tasks

- Trace a RAG pipeline end-to-end with nested observations
- Capture token usage and cost for every LLM call
- Version and deploy prompts via production labels
- Run LLM-as-a-judge evaluations on production traces
- Build datasets from real production traces
- Group multi-turn traces into sessions per user
- Compare model performance across experiments
- Self-host an open-source LLM observability stack
- Instrument OpenAI calls with a drop-in wrapper
- Send OpenTelemetry traces from any framework to Langfuse
- Annotate traces manually with human reviewers
- Track prompt performance across versions

### User Intent Phrases

- How do I debug a RAG pipeline that returns wrong answers?
- I need open-source observability for my LLM application.
- How can I version and A/B test prompts without redeploying code?
- I want to track token cost per user and per session.
- Show me an alternative to LangSmith that I can self-host.
- How do I run LLM-as-a-judge evaluations on my traces?
- I want to build evaluation datasets from production logs.
- How do I monitor LLM quality continuously in production?
- I need OpenTelemetry-native LLM tracing.
- How can I capture inputs and outputs of every LLM call automatically?
- I want to compare GPT-4o vs GPT-4o-mini using experiments.
- How do I annotate traces with human feedback?

### Problem Statements

- General-purpose observability tools lack LLM-specific concepts (tokens, prompts, generations).
- Prompts are buried in code and require redeploys to change.
- We have no idea which prompt version performs best in production.
- Evaluating LLM quality at scale requires automated judges.
- Multi-turn agent workflows are opaque without session grouping.
- LLM costs are surging and we cannot attribute them per user or feature.
- SaaS observability vendors require sending sensitive prompts off-premise.

### When to Pick This

- Pick this when you want a fully open-source, self-hostable LLM observability stack under MIT license.
- Pick this over LangSmith when you need on-prem data residency or want to avoid vendor lock-in.
- Pick this over Arize Phoenix when you need integrated prompt management and experiments as first-class features (not only tracing).
- Pick this over Helicone when you need full evaluation, dataset, and annotation workflows beyond gateway-style request logging.
- Pick this over Weights & Biases when LLM observability — not ML experiment tracking — is the primary need.
- Pick this when your application already uses OpenTelemetry and you want LLM-aware semantic conventions on top.

### Related Terms and Aliases

- LLM ops platform
- LLM application monitoring
- generation observation
- trace tree
- prompt hub
- experiment runner
- LFM (Langfuse managed)
- @observe Python decorator
- LangfuseSpanProcessor
- OTLP exporter
- Langfuse Cloud (EU/US)

