# Arize Phoenix

> Open-source AI observability for debugging and iterating on LLM applications

| Field | Value |
|-------|-------|
| Group | Observability & LLM Ops |
| Type | SDK/UI |
| Open Source | Yes |
| GitHub | [Arize-AI/phoenix](https://github.com/Arize-AI/phoenix) |
| Stars | 8628 |
| Documentation | [Official Docs](https://arize.com/docs/phoenix) |

## Overview

Arize Phoenix is an open-source AI observability and evaluation platform designed for experimentation, evaluation, and troubleshooting of Large Language Model (LLM) applications. Built on OpenTelemetry standards and powered by OpenInference instrumentation, Phoenix provides a vendor-agnostic and language-agnostic approach to capturing, analyzing, and improving AI application behavior. [1][2]

The platform is organized around four core workflows: **tracing** captures the full execution path of LLM applications through distributed traces; **evaluation** measures output quality using LLM-based, code-based, and human evaluators; **prompt engineering** enables iterative prompt improvement with version control and playground experimentation; and **datasets and experiments** provide systematic comparison of application versions against consistent inputs and evaluation criteria. [1]

Phoenix runs as a standalone server with a web UI for visualization and analysis, backed by either SQLite (default) or PostgreSQL for persistent storage. It accepts traces via the OpenTelemetry Protocol (OTLP), making it compatible with any OpenTelemetry-instrumented application. SDKs are available for Python, TypeScript, and Java. [1][2]

## Core Concepts

**Traces** represent the complete execution path of a single request through an LLM application. A trace captures the sequence of operations from initial input to final output, including model calls, document retrieval, tool invocations, and custom logic. Phoenix accepts traces via OTLP over HTTP or gRPC. [1]

**Spans** are the individual units within a trace, each representing a discrete operation. Phoenix defines nine span kinds to categorize operations: LLM (model API calls), EMBEDDING (vector generation), CHAIN (orchestration steps), RETRIEVER (data fetching), RERANKER (document relevance scoring), TOOL (external tool invocations), AGENT (reasoning blocks coordinating tools), GUARDRAIL (safety filtering), and EVALUATOR (output assessment). [3]

**Projects** organize traces by application or environment. Each project collects its own set of traces, evaluations, and annotations, allowing teams to separate development, staging, and production data within a single Phoenix instance. [1]

**Evaluations** are quality assessments attached to traces or spans. Phoenix supports three evaluation methods: LLM-based evaluators that use language models to judge output quality, code-based evaluators that apply deterministic checks (regex, exact match, distance metrics), and human annotations with ground truth labels. [1][4]

**Datasets** are versioned collections of input-output examples extracted from traces or uploaded directly. They serve as fixed benchmarks for running experiments and comparing application changes. [1]

**Experiments** systematically compare application versions by running them against the same dataset with identical evaluators. Each experiment records task outputs and evaluator scores, enabling side-by-side comparison of different prompts, models, or retrieval configurations. [5]

**Prompts** are managed artifacts with version control, supporting Mustache template syntax (`{{ variable }}`) and f-string formatting. Each prompt version records its template content, model configuration, and associated metadata, enabling teams to track changes and deploy the best-performing version. [6]

**Annotations** are metadata labels attached to spans, including scores, labels, and explanations from human reviewers or automated evaluators. Annotations provide the ground truth data used to measure and improve application quality. [1]

## Installation and Setup

### Python Installation

Install the Phoenix server and OpenTelemetry integration:

```bash
pip install arize-phoenix
pip install arize-phoenix-otel
```

For evaluation capabilities, install the evals package:

```bash
pip install arize-phoenix-evals
```

For client-only usage (connecting to an existing Phoenix server):

```bash
pip install arize-phoenix-client
```

### TypeScript Installation

```bash
npm install @arizeai/phoenix-otel @arizeai/openinference-core
npm install @arizeai/phoenix-client    # Client SDK
npm install @arizeai/phoenix-evals     # Evaluations
```

### Launching the Phoenix Server

Start Phoenix locally in a terminal:

```bash
phoenix serve
```

This launches the server on `http://localhost:6006` with the gRPC OTLP collector on port 4317. The web UI is immediately accessible for viewing traces and running evaluations. [1]

### Docker Deployment

```bash
docker run -p 6006:6006 -p 4317:4317 arizephoenix/phoenix:latest
```

For production, pin to a specific version:

```bash
docker run -p 6006:6006 -p 4317:4317 arizephoenix/phoenix:version-8.0.0
```

### Connecting and Instrumenting

Configure the tracer provider to send traces to Phoenix:

```python
from phoenix.otel import register

tracer_provider = register(
    project_name="my-llm-app",
    auto_instrument=True,
)
tracer = tracer_provider.get_tracer(__name__)
```

With `auto_instrument=True`, Phoenix automatically discovers and activates all installed OpenInference instrumentor packages. [3]

For explicit instrumentation of specific libraries:

```python
from openinference.instrumentation.openai import OpenAIInstrumentor
from phoenix.otel import register

tracer_provider = register()
OpenAIInstrumentor().instrument(tracer_provider=tracer_provider)
```

### Environment Variables

Set the connection endpoint and authentication:

```bash
export PHOENIX_COLLECTOR_ENDPOINT="http://localhost:6006"
export PHOENIX_API_KEY="your-phoenix-api-key"   # Required for cloud or authenticated instances
```

## Architecture

Phoenix follows a collector-server-UI architecture built on OpenTelemetry:

```
Application Code
    |
    | (OTLP over HTTP/gRPC)
    v
Phoenix Collector (port 4317 gRPC, port 6006 HTTP)
    |
    v
Storage Backend (SQLite or PostgreSQL)
    |
    v
Phoenix Web UI (port 6006)
    |
    v
REST API / Python Client / TypeScript Client
```

**Collector Layer**: Accepts traces via OTLP, the standard OpenTelemetry protocol. Any OpenTelemetry-compatible application can send traces to Phoenix without Phoenix-specific dependencies. The gRPC collector runs on port 4317 and HTTP on port 6006 by default. [1]

**Storage Layer**: Phoenix stores traces, evaluations, datasets, and experiments in a SQL database. SQLite is the default for local development (stored in `~/.phoenix/`). PostgreSQL is supported for production deployments with higher concurrency and durability requirements. [7]

**Web UI**: A browser-based interface for exploring traces, viewing span details, running evaluations, managing prompts, and comparing experiments. The UI provides filtering, search, and visualization of trace hierarchies with timing and token usage breakdowns. [1]

**Client SDK**: The `arize-phoenix-client` package provides a REST API client for programmatic access to all Phoenix resources: projects, traces, spans, annotations, datasets, experiments, and prompts. Both synchronous (`Client`) and asynchronous (`AsyncClient`) implementations are available. [8]

**OpenInference**: An open-source project maintained alongside Phoenix that defines semantic conventions for AI observability on top of OpenTelemetry. OpenInference provides auto-instrumentor packages for popular frameworks and the span attribute schema that Phoenix uses to parse and display trace data. [1]

### Python Packages

| Package | Purpose |
|---------|---------|
| `arize-phoenix` | Full Phoenix server with UI |
| `arize-phoenix-otel` | Lightweight OpenTelemetry wrapper with Phoenix defaults |
| `arize-phoenix-client` | REST API client for server interaction |
| `arize-phoenix-evals` | Evaluation framework (LLM-based and code-based) |

### TypeScript Packages

| Package | Purpose |
|---------|---------|
| `@arizeai/phoenix-otel` | OpenTelemetry configuration for Phoenix |
| `@arizeai/phoenix-client` | REST API client |
| `@arizeai/phoenix-evals` | Evaluation framework |
| `@arizeai/openinference-core` | Core instrumentation utilities |

## Key Features and Functionality

### Distributed Tracing

Phoenix captures the full execution path of LLM applications through OpenTelemetry-based distributed tracing. Auto-instrumentation packages automatically capture spans for supported frameworks without code changes. The trace view provides visibility into performance bottlenecks, token usage breakdown across LLM calls, runtime exceptions, retrieved documents with relevance scores, LLM parameters and prompt templates, and function call selections. [1]

```python
from phoenix.otel import register

tracer_provider = register(
    project_name="my-rag-app",
    auto_instrument=True,
)
```

### Manual Instrumentation

For custom logic not covered by auto-instrumentation, Phoenix provides decorators and context managers:

```python
from phoenix.otel import register

tracer_provider = register(project_name="my-app")
tracer = tracer_provider.get_tracer(__name__)

@tracer.chain
def my_processing_step(input: str) -> str:
    return process(input)

@tracer.tool
def search_database(query: str) -> str:
    """Search the product database."""
    return results

@tracer.agent
def run_agent(input: str) -> str:
    return agent_response
```

Context managers provide granular control over individual code blocks:

```python
with tracer.start_as_current_span("custom-step", openinference_span_kind="chain") as span:
    span.set_input("input data")
    result = do_work()
    span.set_output(result)
```

LLM spans support rich metadata including message history, token counts, and model parameters:

```python
@tracer.llm(
    process_input=process_input,
    process_output=process_output,
)
def invoke_llm(messages):
    return client.chat.completions.create(messages=messages)
```

### Context Attributes

Attach metadata to traces using context managers that propagate to all child spans:

```python
from openinference.instrumentation import using_attributes

with using_attributes(
    session_id="session-123",
    user_id="user-456",
    metadata={"environment": "production"},
    tags=["v2", "experiment-a"],
    prompt_template="Summarize: {{ text }}",
    prompt_template_version="v1.0",
):
    result = my_llm_chain(input_text)
```

Individual context managers are also available for setting specific attributes:

```python
from openinference.instrumentation import using_session, using_user, using_metadata, using_tags

with using_session(session_id="session-123"):
    with using_user("user-456"):
        result = my_llm_chain(input_text)
```

### Evaluations

Phoenix provides a flexible evaluation framework for measuring output quality. Evaluators can be LLM-based (using language models as judges) or code-based (deterministic checks): [4]

```python
from phoenix.evals import create_classifier
from phoenix.evals.llm import LLM

llm = LLM(provider="openai", model="gpt-4o")

helpfulness_evaluator = create_classifier(
    name="helpfulness",
    prompt_template="Rate the response as helpful or not:\n\nQuery: {input}\nResponse: {output}",
    llm=llm,
    choices={"helpful": 1.0, "not_helpful": 0.0},
)

scores = helpfulness_evaluator.evaluate({
    "input": "How do I reset my password?",
    "output": "Go to Settings > Security > Reset Password.",
})
```

Run evaluations across DataFrames of traces:

```python
from phoenix.evals.evaluators import async_evaluate_dataframe

results_df = await async_evaluate_dataframe(
    dataframe=trace_df,
    evaluators=[helpfulness_evaluator, relevance_evaluator],
)
```

### Datasets and Experiments

Create datasets from traces or direct uploads, then run experiments to compare application versions: [5]

```python
from phoenix.client import Client
from phoenix.client.experiments import run_experiment

client = Client()

dataset = client.datasets.get_dataset(dataset="my-test-set")

def task(input):
    return my_llm_chain(input["question"])

def evaluate_correctness(output, expected):
    return output.strip().lower() == expected.strip().lower()

experiment = run_experiment(
    dataset,
    task=task,
    evaluators=[evaluate_correctness],
    experiment_name="v2-prompt-update",
)
```

Experiments support both synchronous and asynchronous task functions. The `dry_run=True` parameter allows testing the setup before full execution. Evaluators can be added or modified after an experiment completes using `evaluate_experiment()`.

### Prompt Management

Store, version, and retrieve prompts programmatically: [6]

```python
from phoenix.client import Client
from phoenix.client.types import PromptVersion

client = Client()

content = """You're an expert educator in {{ topic }}.
Summarize the following article in bullet points:
{{ article }}"""

prompt = client.prompts.create(
    name="article-summarizer",
    prompt_description="Summarize articles for beginners",
    version=PromptVersion(
        [{"role": "user", "content": content}],
        model_name="gpt-4o-mini",
    ),
)
```

Retrieve prompts by name (latest version), specific version ID, or tag:

```python
prompt = client.prompts.get(name="article-summarizer")
```

### Prompt Playground

The web UI includes an interactive Prompt Playground for iterating on prompts with production data. Features include side-by-side model comparison, parameter adjustment (temperature, max tokens), span replay to test prompts against real inputs captured in traces, and multi-variant testing across model providers. [1]

### Trace Suppression

Conditionally disable tracing for sensitive or high-frequency operations:

```python
from phoenix.otel import suppress_tracing

with suppress_tracing():
    sensitive_result = call_internal_service()
```

## Use Cases

**RAG Application Debugging**: Trace the full retrieval-augmented generation pipeline to identify retrieval failures, irrelevant document selection, or hallucinations. Inspect retrieved documents with relevance scores, view embedding distances, and evaluate response quality against ground truth.

**Agent Loop Analysis**: Visualize multi-step agent executions to identify infinite loops, unnecessary tool calls, or suboptimal tool selection patterns. Agent spans capture the reasoning chain including which tools were called, with what parameters, and in what order.

**Production Monitoring**: Deploy Phoenix as a persistent server receiving traces from production applications. Set up automated evaluators to score incoming traces, track quality metrics over time, and set alerts on degradation patterns.

**Model Comparison**: Use the Prompt Playground to compare responses across different models (GPT-4o, Claude, Gemini) with identical inputs. Run experiments against datasets to quantify performance differences with statistical rigor.

**Prompt Iteration**: Capture production traces to build datasets of real-world inputs, then iterate on prompts using the Playground with span replay. Version each prompt change and run experiments to validate improvements before deployment.

**Evaluation Pipeline Development**: Build custom evaluation pipelines combining LLM judges, code-based checks, and human annotations. Use benchmark datasets to calibrate evaluator accuracy before deploying them for automated scoring.

**Cost Optimization**: Break down token usage across LLM calls within traces to identify expensive operations. Compare token consumption across prompt versions or model choices to optimize cost-performance tradeoffs.

## API Reference Summary

### Python Client (`arize-phoenix-client`)

**Client Initialization**:

```python
from phoenix.client import Client

client = Client()                                    # Local server (localhost:6006)
client = Client(endpoint="https://your-phoenix.com") # Remote server
```

**Prompts Resource**:
- `client.prompts.create(name, version, prompt_description)` -- Create a new prompt with initial version
- `client.prompts.get(name, version_id, tag)` -- Retrieve prompt by name, version, or tag

**Datasets Resource**:
- `client.datasets.list()` -- List all datasets
- `client.datasets.get_dataset(dataset)` -- Fetch dataset with examples
- `client.datasets.create_dataset(name, inputs, outputs)` -- Create dataset from dictionaries, DataFrames, or CSV
- `client.datasets.to_dataframe(dataset)` -- Convert dataset to pandas DataFrame

**Spans Resource**:
- `client.spans.get_spans_dataframe(project_name, filter)` -- Query spans as DataFrame with filtering
- `client.spans.get_span_annotations_dataframe(project_name)` -- Retrieve span annotations
- `client.spans.log_span_annotations(annotations)` -- Bulk log annotations

**Projects Resource**:
- `client.projects.list()` -- List all projects
- `client.projects.create(name)` -- Create a new project

**Experiments**:
- `run_experiment(dataset, task, evaluators, experiment_name)` -- Run experiment against dataset
- `evaluate_experiment(experiment, evaluators)` -- Add evaluators to completed experiment

### REST API Endpoints

Phoenix exposes a REST API for spans, annotations, datasets, and experiments. The `/v1/traces` endpoint accepts OTLP trace data via HTTP POST. [8]

### OpenTelemetry Integration

Phoenix accepts standard OTLP traces on:
- **HTTP**: `http://localhost:6006/v1/traces`
- **gRPC**: `localhost:4317`

## Configuration and Customization

### Server Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PHOENIX_PORT` | 6006 | Web server and HTTP collector port |
| `PHOENIX_GRPC_PORT` | 4317 | gRPC OTLP collector port |
| `PHOENIX_HOST` | 0.0.0.0 | Server host binding |
| `PHOENIX_WORKING_DIR` | ~/.phoenix/ | Data directory for SQLite and exports |
| `PHOENIX_SQL_DATABASE_URL` | SQLite | Database connection string (PostgreSQL or SQLite) |
| `PHOENIX_HOST_ROOT_PATH` | (none) | Root path prefix for reverse proxy deployments |
| `PHOENIX_DEFAULT_RETENTION_POLICY_DAYS` | 0 (infinite) | Trace retention period in days |
| `PHOENIX_ENABLE_PROMETHEUS` | false | Enable Prometheus metrics on port 9090 |
| `PHOENIX_CSRF_TRUSTED_ORIGINS` | (none) | Comma-separated origins bypassing CSRF protection |
| `PHOENIX_ALLOW_EXTERNAL_RESOURCES` | true | External resource loading (set false for air-gapped) |
| `PHOENIX_TELEMETRY_ENABLED` | true | Anonymous usage analytics (never collects trace data) |
| `PHOENIX_DEFAULT_ADMIN_INITIAL_PASSWORD` | admin | Initial admin password when authentication is enabled |

### PostgreSQL Configuration

For production deployments, configure PostgreSQL either via connection string or individual variables: [7]

```bash
export PHOENIX_SQL_DATABASE_URL="postgresql://user:pass@localhost:5432/phoenix"

# Or via individual variables:
export PHOENIX_POSTGRES_HOST="localhost"
export PHOENIX_POSTGRES_PORT="5432"
export PHOENIX_POSTGRES_USER="phoenix"
export PHOENIX_POSTGRES_PASSWORD="secret"
export PHOENIX_POSTGRES_DB="phoenix"
export PHOENIX_SQL_DATABASE_SCHEMA="phoenix"  # Optional schema
```

### Docker Compose with PostgreSQL

```yaml
version: "3.8"
services:
  phoenix:
    image: arizephoenix/phoenix:version-8.0.0
    ports:
      - "6006:6006"
      - "4317:4317"
    environment:
      - PHOENIX_SQL_DATABASE_URL=postgresql://phoenix:secret@db:5432/phoenix
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      - POSTGRES_USER=phoenix
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=phoenix
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

### Authentication

Phoenix supports authentication via OAuth2, Lightweight Directory Access Protocol (LDAP), and local accounts with role-based access controls. When authentication is enabled, all API interactions require an `Authorization: Bearer <token>` header or the `PHOENIX_API_KEY` environment variable. [7]

### Client Configuration

```python
import os

os.environ["PHOENIX_COLLECTOR_ENDPOINT"] = "https://your-phoenix-instance.com"
os.environ["PHOENIX_API_KEY"] = "your-api-key"
os.environ["PHOENIX_CLIENT_HEADERS"] = "api_key=your-api-key"
```

## Integration Patterns

### With OpenAI

```python
from openinference.instrumentation.openai import OpenAIInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name="openai-app")
OpenAIInstrumentor().instrument(tracer_provider=tracer_provider)

# All OpenAI calls are now automatically traced
from openai import OpenAI
client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}],
)
```

### With LangChain

```python
from openinference.instrumentation.langchain import LangChainInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name="langchain-app")
LangChainInstrumentor().instrument(tracer_provider=tracer_provider)
```

### With LlamaIndex

```python
from openinference.instrumentation.llama_index import LlamaIndexInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name="llamaindex-app")
LlamaIndexInstrumentor().instrument(tracer_provider=tracer_provider)
```

### With Haystack

```python
from openinference.instrumentation.haystack import HaystackInstrumentor
from phoenix.otel import register

tracer_provider = register()
HaystackInstrumentor().instrument(tracer_provider=tracer_provider)
```

### With CrewAI

```python
from openinference.instrumentation.crewai import CrewAIInstrumentor
from openinference.instrumentation.openai import OpenAIInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name="crewai-app", auto_instrument=True)
```

### With DSPy

```python
from openinference.instrumentation.dspy import DSPyInstrumentor
from phoenix.otel import register

tracer_provider = register(project_name="dspy-app")
DSPyInstrumentor().instrument(tracer_provider=tracer_provider)
```

### With Evaluation Frameworks (Ragas, Deepeval, Cleanlab)

Phoenix integrates with third-party evaluation frameworks. Trace evaluations from Ragas, Deepeval, or Cleanlab can be ingested as annotations on Phoenix spans, combining framework-specific metrics with Phoenix's trace visualization. [1]

### TypeScript / Vercel AI SDK

```typescript
import { NodeTracerProvider } from "@opentelemetry/sdk-trace-node";
import { BatchSpanProcessor } from "@opentelemetry/sdk-trace-base";
import { OTLPTraceExporter } from "@opentelemetry/exporter-trace-otlp-proto";
import { resourceFromAttributes } from "@opentelemetry/resources";
import { ATTR_SERVICE_NAME } from "@opentelemetry/semantic-conventions";
import { SEMRESATTRS_PROJECT_NAME } from "@arizeai/openinference-semantic-conventions";

const provider = new NodeTracerProvider({
    resource: resourceFromAttributes({
        [ATTR_SERVICE_NAME]: "my-app",
        [SEMRESATTRS_PROJECT_NAME]: "my-app",
    }),
    spanProcessors: [
        new BatchSpanProcessor(
            new OTLPTraceExporter({
                url: `${process.env.PHOENIX_COLLECTOR_ENDPOINT}/v1/traces`,
            })
        ),
    ],
});

provider.register();
```

### With LiteLLM Proxy

Phoenix integrates with LiteLLM as an observability backend. LiteLLM can be configured to send traces to Phoenix, providing visibility across all models routed through the proxy.

## Examples

### End-to-End RAG Tracing

```python
from openinference.instrumentation.openai import OpenAIInstrumentor
from phoenix.otel import register

tracer_provider = register(
    project_name="rag-pipeline",
    endpoint="http://localhost:6006/v1/traces",
    auto_instrument=True,
)
tracer = tracer_provider.get_tracer(__name__)

from openai import OpenAI
client = OpenAI()

@tracer.chain
def rag_pipeline(question: str) -> str:
    documents = retrieve_documents(question)
    context = "\n".join(doc.text for doc in documents)
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": f"Answer using this context:\n{context}"},
            {"role": "user", "content": question},
        ],
    )
    return response.choices[0].message.content

@tracer.retriever
def retrieve_documents(query: str):
    # Vector search implementation
    return vector_store.similarity_search(query, k=5)
```

### Custom LLM Evaluator with Benchmark Dataset

```python
from phoenix.client import Client
from phoenix.client.experiments import run_experiment
from phoenix.evals import create_classifier
from phoenix.evals.llm import LLM

client = Client()
llm = LLM(provider="openai", model="gpt-4o-mini")

tone_evaluator = create_classifier(
    name="tone",
    llm=llm,
    prompt_template="Is the tone of this response professional?\n\nResponse: {output}",
    choices={"professional": 1.0, "unprofessional": 0.0},
)

relevance_evaluator = create_classifier(
    name="relevance",
    llm=llm,
    prompt_template="Is this response relevant to the query?\n\nQuery: {input}\nResponse: {output}",
    choices={"relevant": 1.0, "irrelevant": 0.0},
)

dataset = client.datasets.get_dataset(dataset="customer-support-test")

experiment = run_experiment(
    dataset,
    task=my_support_agent,
    evaluators=[tone_evaluator, relevance_evaluator],
    experiment_name="support-agent-v3",
)
```

### Prompt Versioning Workflow

```python
from phoenix.client import Client
from phoenix.client.types import PromptVersion

client = Client()

# Create initial prompt version
v1_content = "Summarize this article:\n{{ article }}"
client.prompts.create(
    name="summarizer",
    version=PromptVersion(
        [{"role": "user", "content": v1_content}],
        model_name="gpt-4o-mini",
    ),
)

# Create improved version
v2_content = """You are an expert technical writer.
Summarize the following article in 3-5 bullet points for a technical audience:
{{ article }}"""

client.prompts.create(
    name="summarizer",
    version=PromptVersion(
        [{"role": "user", "content": v2_content}],
        model_name="gpt-4o",
    ),
)

# Retrieve latest version in production
prompt = client.prompts.get(name="summarizer")
```

### Session-Aware Tracing

```python
from openinference.instrumentation import using_attributes
from phoenix.otel import register

register(project_name="chatbot", auto_instrument=True)

def handle_message(session_id: str, user_id: str, message: str) -> str:
    with using_attributes(
        session_id=session_id,
        user_id=user_id,
        metadata={"source": "web-chat"},
        tags=["production", "v2"],
    ):
        return chatbot.respond(message)
```

## Limitations and Considerations

**Product Distinction**: Arize offers two separate products -- Phoenix (open-source, self-hosted) and Arize AX (commercial cloud platform). These have different APIs, authentication methods, and endpoints. Documentation and SDK imports differ between the two; verify which product you are using before following guides. [1]

**Storage Scalability**: The default SQLite backend is suitable for development and small-scale deployments. Production environments with high trace volumes or concurrent users should use PostgreSQL for better performance and reliability. [7]

**Evaluation Latency**: LLM-based evaluators introduce latency and cost proportional to the number of spans evaluated. For high-throughput applications, consider sampling strategies or asynchronous evaluation pipelines rather than evaluating every trace.

**Auto-Instrumentation Scope**: Auto-instrumentation captures framework-level operations but cannot instrument custom business logic. Applications with significant custom processing between framework calls require manual instrumentation with decorators or context managers to achieve complete trace coverage. [3]

**Local Server Resources**: Running Phoenix locally with `phoenix serve` stores all data in `~/.phoenix/` using SQLite. For long-running production use, the working directory can grow substantially; configure `PHOENIX_DEFAULT_RETENTION_POLICY_DAYS` to manage data lifecycle. [7]

**TypeScript Parity**: While Phoenix provides Python and TypeScript SDKs, the Python ecosystem has broader auto-instrumentation coverage and more mature evaluation tooling. TypeScript support relies on standard OpenTelemetry configuration rather than the streamlined `register()` API available in Python.

**Cold Start**: The Phoenix server requires initialization time to load the database and start the web UI. In containerized environments, account for this startup delay in health check configurations.

## Changelog Highlights

- **OpenInference Instrumentation**: Expanded auto-instrumentation support for OpenAI (including Agents SDK), Anthropic, CrewAI, Pydantic AI, Autogen, Mastra, and Vercel AI SDK
- **Prompt Management**: Version-controlled prompt storage with Mustache and f-string template support, plus SDK-based retrieval for production deployment
- **Experiments Framework**: Systematic comparison of application versions with dataset-driven evaluation and side-by-side result analysis
- **Nine Span Kinds**: Expanded from initial span types to include AGENT, GUARDRAIL, and EVALUATOR span categories
- **PostgreSQL Support**: Production-grade database backend as alternative to SQLite
- **Authentication and RBAC**: OAuth2, LDAP, and local account authentication with role-based access controls
- **Prometheus Metrics**: Optional metrics export for integration with existing monitoring infrastructure
- **Docker Non-Root Images**: Security-hardened container images (`phoenix:latest-nonroot`) for production Kubernetes deployments [1][2]

## Citations

- [1] Phoenix Documentation - <https://arize.com/docs/phoenix>
- [2] Phoenix GitHub Repository - <https://github.com/Arize-AI/phoenix>
- [3] Manual Instrumentation Guide - <https://arize.com/docs/phoenix/tracing/how-to-tracing/setup-tracing/instrument>
- [4] Building Custom Evaluators - <https://arize.com/docs/phoenix/evaluation/concepts-evals/building-your-own-evals>
- [5] Running Experiments - <https://arize.com/docs/phoenix/datasets-and-experiments/how-to-experiments/run-experiments>
- [6] Prompt Management - <https://arize.com/docs/phoenix/prompt-engineering/overview-prompts/prompt-management>
- [7] Self-Hosting Configuration - <https://arize.com/docs/phoenix/self-hosting/configuration>
- [8] Python Client SDK - <https://arize.com/docs/phoenix/sdk-api-reference/python/arize-phoenix-client>
